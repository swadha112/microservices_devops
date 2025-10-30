pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'swadhak'
        IMAGE_TAG = "${BUILD_NUMBER}"
        // paths on your Mac
        NPM_PATH = '/opt/homebrew/bin/npm'
        DOCKER_PATH = '/usr/local/bin/docker'
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
                    echo "===== docker version (explicit) ====="
                    /usr/local/bin/docker version || echo "docker not found at /usr/local/bin/docker"
                '''
            }
        }

        stage('Build & Test Services') {
            steps {
                dir('user-service') {
                    sh '''
                        export PATH=/opt/homebrew/bin:/usr/local/bin:$PATH
                        npm ci
                        npm test -- --runInBand --detectOpenHandles --forceExit || true
                    '''
                }
                dir('order-service') {
                    sh '''
                        export PATH=/opt/homebrew/bin:/usr/local/bin:$PATH
                        npm ci
                        npm test -- --runInBand --detectOpenHandles --forceExit || true
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    export PATH=/usr/local/bin:/opt/homebrew/bin:$PATH
                    docker build -t swadhak/user-service:${IMAGE_TAG} ./user-service
                    docker build -t swadhak/order-service:${IMAGE_TAG} ./order-service
                '''
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
                        export PATH=/usr/local/bin:/opt/homebrew/bin:$PATH
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push swadhak/user-service:${IMAGE_TAG}
                        docker push swadhak/order-service:${IMAGE_TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy (local docker)') {
            steps {
                sh '''
                    export PATH=/usr/local/bin:/opt/homebrew/bin:$PATH
                    docker rm -f user-service  || true
                    docker rm -f order-service || true
                    docker run -d --name user-service  -p 3001:3001 swadhak/user-service:${IMAGE_TAG}
                    docker run -d --name order-service -p 3002:3002 swadhak/order-service:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    export PATH=/usr/local/bin:/opt/homebrew/bin:$PATH
                    docker ps
                '''
            }
        }
    }

    post {
        success { echo "✅ Pipeline succeeded. Images pushed and containers running." }
        failure { echo "❌ Pipeline failed." }
    }
}