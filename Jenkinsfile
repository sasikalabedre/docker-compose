pipeline {
    agent any

    environment {
        FRONTEND_IMAGE = 'sasikalabedre/docker-compose-frontend:latest'
        BACKEND_IMAGE  = 'sasikalabedre/docker-compose-backend:latest'
    }

    stages {

        // 1. Get latest code from GitHub
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // 2. Build Frontend and Backend Docker images
        stage('Build Docker Images') {
            steps {
                sh '''
                    echo "Building Frontend image..."
                    docker build -t $FRONTEND_IMAGE ./react

                    echo "Building Backend image..."
                    docker build -t $BACKEND_IMAGE ./node

                    echo "Docker images built successfully."
                    docker images
                '''
            }
        }

        // 3. Login to Docker Hub
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        // 4. Push images to Docker Hub
        stage('Push Images to Docker Hub') {
            steps {
                sh '''
                    echo "Pushing Frontend image..."
                    docker push $FRONTEND_IMAGE

                    echo "Pushing Backend image..."
                    docker push $BACKEND_IMAGE
                '''
            }
        }

        // 5. Remove only old Frontend and Backend containers
        stage('Remove Old Frontend and Backend') {
            steps {
                sh '''
                    echo "Removing old Frontend container..."
                    docker rm -f frontend || true

                    echo "Removing old Backend container..."
                    docker rm -f backend || true

                    echo "Old Frontend and Backend containers removed."
                '''
            }
        }

        // 6. Pull latest images from Docker Hub
        stage('Pull Latest Images') {
            steps {
                sh '''
                    echo "Pulling latest Frontend image..."
                    docker pull $FRONTEND_IMAGE

                    echo "Pulling latest Backend image..."
                    docker pull $BACKEND_IMAGE
                '''
            }
        }

        // 7. Start only Frontend and Backend
        stage('Deploy Frontend and Backend') {
            steps {
                sh '''
                    cd /home/sasikala_bedre_gmail_com/dockerize-react-node-postgres-nginx-application

                    echo "Starting Frontend and Backend..."

                    docker compose up -d frontend backend

                    echo "Deployment completed."
                '''
            }
        }

        // 8. Verify containers
        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Checking running containers..."
                    docker ps

                    echo "Checking Frontend..."
                    docker ps --filter "name=frontend"

                    echo "Checking Backend..."
                    docker ps --filter "name=backend"

                    echo "Checking Database..."
                    docker ps --filter "name=db"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD deployment completed successfully!'
        }

        failure {
            echo 'CI/CD deployment failed!'
        }
    }
}
