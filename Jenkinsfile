pipeline {
    agent any

    environment {
        IMAGE_NAME = "mfblessed078/myportfolio"
        IMAGE_TAG =  "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'

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
                echo '========== STAGE 3: Pushing image to Docker Hub =========='

                script {
                    docker.withRegistry('', "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push('latest')
                    }

                    echo "Image pushed to Docker Hub: ${IMAGE_NAME}:${IMAGE_TAG} and :latest"
                }
            }
        }



        stage('Deploy') {
            steps {
                echo '========== STAGE 4: Deploying to Kubernetes =========='

		sh  'export KUBECONFIG=/var/lib/jenkins/.kube/config'
               
                 sh 'kubectl apply -f k8s/deployment.yaml'

                sh 'kubectl apply -f k8s/service.yaml'

                sh 'kubectl rollout restart deployment/myportfolio'


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
            sh "docker rmi ${IMAGE_NAME}:latest || true"
            echo 'Workspace cleaned up.'
        }
    }

}

