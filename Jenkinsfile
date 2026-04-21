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
                git branch: 'main', url: 'https://github.com/kamalateck/vm-pipeline.git'
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
        withCredentials([sshUserPrivateKey(credentialsId: 'vm-ssh-key',
                                          keyFileVariable: 'KEY',
                                          usernameVariable: 'USER')]) {

            sh """
            ssh -o StrictHostKeyChecking=no -i $KEY $USER@34.171.135.166 '
                echo "Stopping old container..."
                docker stop vm-container || true
                docker rm vm-container || true

                echo "Pulling new image..."
                docker pull kamalteck/vmimage:${BUILD_NUMBER}

                echo "Running new container..."
                docker run -d -p 8000:8000 --name vm-container kamalteck/vmimage:${BUILD_NUMBER}
            '
            """
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
