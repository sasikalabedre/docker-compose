pipeline {
    agent any

    environment {
        FRONTEND_IMAGE = 'sasikalabedre/docker-compose-frontend:latest'
        BACKEND_IMAGE  = 'sasikalabedre/docker-compose-backend:latest'
        DOCKER_NETWORK = 'dockerize-react-node-postgres-nginx-application_node-network'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    echo "===== Building Frontend Image ====="
                    docker build -t $FRONTEND_IMAGE ./react

                    echo "===== Building Backend Image ====="
                    docker build -t $BACKEND_IMAGE ./node

                    echo "===== Docker Images ====="
                    docker images
                '''
            }
        }

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

        stage('Push Images to Docker Hub') {
            steps {
                sh '''
                    echo "===== Pushing Frontend Image ====="
                    docker push $FRONTEND_IMAGE

                    echo "===== Pushing Backend Image ====="
                    docker push $BACKEND_IMAGE
                '''
            }
        }

        stage('Remove Old Frontend and Backend') {
            steps {
                sh '''
                    echo "===== Removing old frontend ====="
                    docker rm -f frontend || true

                    echo "===== Removing old backend ====="
                    docker rm -f backend || true

                    echo "Database container is NOT touched."
                '''
            }
        }

        stage('Pull Latest Images') {
            steps {
                sh '''
                    echo "===== Pulling latest frontend ====="
                    docker pull $FRONTEND_IMAGE

                    echo "===== Pulling latest backend ====="
                    docker pull $BACKEND_IMAGE
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh '''
                    echo "===== Starting Frontend ====="

                    docker run -d \
                        --name frontend \
                        --network $DOCKER_NETWORK \
                        -p 5173:5173 \
                        -w /usr/src/app \
                        $FRONTEND_IMAGE \
                        npm run dev
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    echo "===== Starting Backend ====="

                    docker run -d \
                        --name backend \
                        --network $DOCKER_NETWORK \
                        -p 3000:3000 \
                        -w /usr/src/app \
                        $BACKEND_IMAGE \
                        npm run start
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "===== All Running Containers ====="
                    docker ps

                    echo "===== Frontend ====="
                    docker ps --filter "name=frontend"

                    echo "===== Backend ====="
                    docker ps --filter "name=backend"

                    echo "===== Database ====="
                    docker ps --filter "name=db"

                    echo "===== Nginx ====="
                    docker ps --filter "name=nginx"
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
