pipeline {
    agent any

    environment {
        PROJECT_ID = 'new-dev-492605'
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-service-account')

        DOCKER_HUB_CREDENTIALS_USR = 'kamalteck'
        IMAGE_NAME = 'vmimage'
        DOCKER_HUB_CREDENTIALS_PSWD = credentials('docker-hub-password')

        VM_USER = 'debian'
        VM_IP = '34.171.135.166'
        CONTAINER_NAME = 'vmimage'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/kamalateck/cloud-run-practice.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Login & Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-password',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {

                        sh """
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push ${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                script {
                    sshagent(['vm-ssh-key']) {

                        sh """
                        ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_IP} '
                            
                            echo "Stopping old container..."
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true

                            echo "Pulling latest image..."
                            docker pull ${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER}

                            echo "Running new container..."
                            docker run -d -p 8000:8000 --name ${CONTAINER_NAME} \
                            ${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER}
                        '
                        """
                    }
                }
            }
        }

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
    }
}