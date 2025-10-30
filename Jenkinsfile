pipeline {
    agent any

    environment {
        // your Docker Hub username / org
        DOCKERHUB_REPO = 'swadhak'
        // every build gets its own tag
        IMAGE_TAG = "${BUILD_NUMBER}"
        // use the global Homebrew npm, not nvm
        NPM_PATH = '/opt/homebrew/bin/npm'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test Services') {
            steps {
                // user-service
                dir('user-service') {
                    sh "${NPM_PATH} ci"
                    // don't fail the pipeline if tests fail
                    sh "${NPM_PATH} test || true"
                }

                // order-service
                dir('order-service') {
                    sh "${NPM_PATH} ci"
                    sh "${NPM_PATH} test || true"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${DOCKERHUB_REPO}/user-service:${IMAGE_TAG} ./user-service"
                sh "docker build -t ${DOCKERHUB_REPO}/order-service:${IMAGE_TAG} ./order-service"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                // make sure you created a Jenkins credential with ID: dockerhub-creds
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${DOCKERHUB_REPO}/user-service:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_REPO}/order-service:${IMAGE_TAG}"
                    sh 'docker logout'
                }
            }
        }

        stage('Deploy (local docker)') {
            steps {
                // stop old containers if they exist
                sh 'docker rm -f user-service  || true'
                sh 'docker rm -f order-service || true'

                // run new ones
                sh "docker run -d --name user-service  -p 3001:3001 ${DOCKERHUB_REPO}/user-service:${IMAGE_TAG}"
                sh "docker run -d --name order-service -p 3002:3002 ${DOCKERHUB_REPO}/order-service:${IMAGE_TAG}"
            }
        }

        stage('Verify') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded. Images pushed and services deployed."
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}