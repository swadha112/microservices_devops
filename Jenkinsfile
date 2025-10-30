pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'swadhak'
        IMAGE_TAG = "${BUILD_NUMBER}"
        HOMEBREW_BIN = '/opt/homebrew/bin'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Show PATH (debug)') {
            steps {
                sh '''
                    echo "===== ENV ====="
                    env | sort
                    echo "===== WHICH NODE/NPM (Homebrew) ====="
                    /opt/homebrew/bin/node -v || echo "node not at /opt/homebrew/bin/node"
                    /opt/homebrew/bin/npm -v || echo "npm not at /opt/homebrew/bin/npm"
                '''
            }
        }

        stage('Build & Test Services') {
            steps {
                dir('user-service') {
                    sh '''
                        export PATH=/opt/homebrew/bin:$PATH
                        echo "PATH in user-service: $PATH"
                        which npm || true
                        npm ci
                        npm test || true
                    '''
                }
                dir('order-service') {
                    sh '''
                        export PATH=/opt/homebrew/bin:$PATH
                        echo "PATH in order-service: $PATH"
                        which npm || true
                        npm ci
                        npm test || true
                    '''
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
                sh 'docker rm -f user-service  || true'
                sh 'docker rm -f order-service || true'
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
            echo "✅ Pipeline succeeded."
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}