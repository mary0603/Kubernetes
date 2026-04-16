pipeline {
    agent any

    environment {
        IMAGE_NAME = "mfblessed078/myportfolio"
        IMAGE_TAG = "v2"
        CONTAINER_NAME = "myportfolio-container"
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {

        stage('Code Pull') {
            steps {
                checkout scm
            }
        }

        stage('Image Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '========== STAGE 4: Deploying to Kubernetes =========='

		sh  'export KUBECONFIG=/var/lib/jenkins/.kube/config'
               
                 sh 'kubectl apply -f k8s/deployment.yaml'

                sh 'kubectl apply -f k8s/service.yaml'

                

                sh 'kubectl rollout status deployment/myportfolio --timeout=300s'

                echo '---------- Deployment Status ----------'
                sh 'kubectl get pods -l app=myportfolio'
                sh 'kubectl get services myportfolio-service'
            }
        }
    }
post {
        success {
            echo '========== PIPELINE COMPLETED SUCCESSFULLY =========='
            echo "Application deployed. Access at http://<EC2_PUBLIC_IP>:30081"
        }
        failure {
            echo '========== PIPELINE FAILED =========='
            echo 'Check the stage logs above for error details.'
        }
        always {
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
            sh "docker rmi ${IMAGE_NAME}:v2 || true"
            echo 'Workspace cleaned up.'
        }
    }

}

