pipeline {
    agent any

    environment {
        IMAGE_NAME = "nagarajkamath602/pipeline-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    tools {
        sonarQube 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/nagaraj602/python-example-app-nag-app-py.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    sonar-scanner
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                docker push $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy Dev') {
            steps {
                sh '''
                docker rm -f dev-container || true

                docker run -d \
                  --name dev-container \
                  -p 5001:5000 \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Approval For Stage') {
            steps {
                input message: 'Deploy to STAGE environment?'
            }
        }

        stage('Deploy Stage') {
            steps {
                sh '''
                docker rm -f stage-container || true

                docker run -d \
                  --name stage-container \
                  -p 5002:5000 \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Approval For Prod') {
            steps {
                input message: 'Deploy to PROD environment?'
            }
        }

        stage('Deploy Prod') {
            steps {
                sh '''
                docker rm -f prod-container || true

                docker run -d \
                  --name prod-container \
                  -p 5003:5000 \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
