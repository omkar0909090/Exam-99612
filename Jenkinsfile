pipeline {
    agent any

    environment {
        IMAGE_NAME = "exam-nginx"
        SERVICE_NAME = "exam-nginx"
        REPLICAS = "3"
        PORT = "5000"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build -t ${IMAGE_NAME}:v${BUILD_NUMBER} .
                '''
            }
        }

        stage('Deploy Docker Service') {
            steps {
                echo 'Deploying Docker service...'

                sh '''
                    if docker service inspect ${SERVICE_NAME} >/dev/null 2>&1
                    then
                        echo "Updating existing service..."

                        docker service update \
                            --image ${IMAGE_NAME}:v${BUILD_NUMBER} \
                            ${SERVICE_NAME}
                    else
                        echo "Creating new service..."

                        docker service create \
                            --name ${SERVICE_NAME} \
                            --replicas ${REPLICAS} \
                            --publish published=${PORT},target=80 \
                            ${IMAGE_NAME}:v${BUILD_NUMBER}
                    fi
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    docker service ls
                    docker service ps ${SERVICE_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }
    }
}
