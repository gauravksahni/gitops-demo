pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "gauravkb"
        APP_NAME = "gitops-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "{DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
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
                git credentialsId: 'github', 
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
    }
}