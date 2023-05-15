pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "gauravkb"
        APP_NAME = "gitops-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
    }
    stages{
        stage('Clean Workspace'){
            steps{
                script {
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM'){
            steps {
                git credentialsId: 'girthub-gitops-api-token', 
                url: 'https://github.com/gauravksahni/gitops-demo.git',
                branch: 'main'  
            }
        }
        stage('Build Docker image'){
            steps{
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage('Push Docker Image'){
            steps{
                script{
                    docker.withRegistry('', REGISTRY_CREDS){
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage('Delete Docker Image'){
            steps{
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker rmi ${IMAGE_NAME}:latest"
            }
        }
        stage('Updating k8s deployment file'){
            steps{
                sh "cat deployment.yml"
                sh "sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml"
                sh "cat deployment.yml"
            }
        }
        stage('Push the changed deployment file to Git'){
            steps {
                script{
                    sh """
                    git config --global user.email "gauravksahni@gmail.com"
                    git config --global user.name "Gaurav"
                    git add deployment.yml
                    git commit -m 'Updated the deployment file' 
                    
                    """
                    withCredentials([gitUsernamePassword(credentialsId: 'girthub-gitops-api-token', gitToolName: 'Default')]) {
                        sh "git push origin main"
                     }
                }
            }
        }
    }
}