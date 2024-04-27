pipeline{
    agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            app: test
        spec:
          containers:
          - name: git
            image: bitnami/git:latest
            command:
            - cat
            tty: true
          - name: sonarcli
            image: sonarsource/sonar-scanner-cli:latest
            command:
            - cat
            tty: true
          - name: curl
            image: alpine/curl:latest
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
            - mountPath: /var/run/docker.sock
              name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock
        '''
        }      
    } 
    environment {
        GITHUB_USERNAME = 'gauravksahni'
        REPOSITORY = 'gitops-demo'
        DOCKERHUB_USERNAME = "gauravkb"
        APP_NAME = 'py-flask-webapp'
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_REGISTRY_TOKEN = credentials('jenkins-dockerhub-integration')
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    tools {
        git 'Default'
    }
    stages{
        // stage('Clean Workspace'){
        //     steps{
        //         script {
        //             cleanWs()
        //         }
        //     }
        // }
        stage('Checkout SCM'){
            steps{
                container('git'){
                    script {
                        println "----------Stage 1 - Checkout Repository------------"
                        checkout scmGit(
                            branches: [[name: '*/main']],
                            extensions: [],
                            userRemoteConfigs: [[url: "https://github.com/${GITHUB_USERNAME}/${REPOSITORY}.git"]]
                        )
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                container('sonarcli'){
                    script {
                        // def scannerHome = tool 'sonarqube-scanner'
                        withSonarQubeEnv('sonarqube-server') {
                            sh '''/opt/sonar-scanner/bin/sonar-scanner \
                                -Dsonar.projectKey=gitops-demo \
                                -Dsonar.sources=. 
                            '''
                        }
                    }
                }
            }
        }
        stage('Wait for Quality Gate'){
            steps{
                container('sonarcli'){
                    timeout(time: 1, unit: 'HOURS') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        stage('Build Docker Image'){
            steps{
                container('docker'){
                    sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
                    sh "docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest"
                }
            }
        }
        stage('Push Docker Image to Docker Hub'){
            steps{
                container('docker'){
                    withCredentials([usernamePassword(credentialsId: 'jenkins-dockerhub-integration', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker push $IMAGE_NAME:$IMAGE_TAG"
                        sh "docker push $IMAGE_NAME:latest"
                    }
                }
            }
        }
    }
