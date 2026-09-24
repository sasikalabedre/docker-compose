pipeline {
    agent any

    environment {
        FRONTEND_IMAGE = 'sasikalabedre/docker-compose-frontend:latest'
        BACKEND_IMAGE  = 'sasikalabedre/docker-compose-backend:latest'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $FRONTEND_IMAGE ./react'
                sh 'docker build -t $BACKEND_IMAGE ./node'
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
                sh 'docker push $FRONTEND_IMAGE'
                sh 'docker push $BACKEND_IMAGE'
            }
        }

        stage('Deploy with Docker Compose') {
    steps {
        sh '''
            cd /home/sasikala_bedre_gmail_com/dockerize-react-node-postgres-nginx-application
            docker compose pull
            docker compose up -d
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
