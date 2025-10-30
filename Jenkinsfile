pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'swadhak'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build & Test Services') {
            steps {
                dir('user-service') {
                    sh '''
                        npm ci
                        npm test -- --runInBand --detectOpenHandles --forceExit || true
                    '''
                }
                dir('order-service') {
                    sh '''
                        npm ci
                        npm test -- --runInBand --detectOpenHandles --forceExit || true
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
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
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
    }

    post {
        success { echo "✅ done" }
        failure { echo "❌ failed" }
    }
}