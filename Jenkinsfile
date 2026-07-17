pipeline {

    agent any


    parameters {

        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'uat'],
            description: 'Select deployment environment'
        )

    }


    environment {

        IMAGE_NAME = 'vmimage'
        CONTAINER_NAME = 'vmimage'

    }



    stages {


        stage('Load Environment Configuration') {

            steps {

                script {


                    if (params.ENVIRONMENT == 'dev') {


                        env.PROJECT_ID = DEV_PROJECT_ID
                        env.IMAGE_PATH = DEV_IMAGE_PATH

                        env.VM_IP = DEV_VM_IP
                        env.VM_USER = DEV_VM_USER

                        env.GCP_CREDENTIAL = "gcp-service-account-dev"

                        env.SSH_CREDENTIAL = "vm-ssh-key"


                    }


                    else {


                        env.PROJECT_ID = UAT_PROJECT_ID
                        env.IMAGE_PATH = UAT_IMAGE_PATH

                        env.VM_IP = UAT_VM_IP
                        env.VM_USER = UAT_VM_USER

                        env.GCP_CREDENTIAL = "gcp-service-account-uat"

                        env.SSH_CREDENTIAL = "vm-uat-ssh-key"


                    }


                    env.IMAGE_TAG = BUILD_NUMBER


                }

            }

        }




        stage('Clone Repository') {

            steps {

                git(
                    branch: 'main',
                    url: 'https://github.com/kamalateck/vm-pipeline.git'
                )

            }

        }





        stage('Build Docker Image') {


            steps {


                sh """

                docker build \
                -t ${IMAGE_PATH}:${IMAGE_TAG} .

                """

            }

        }






        stage('Authenticate GCP') {


            steps {


                withCredentials([

                    file(
                        credentialsId: "${GCP_CREDENTIAL}",
                        variable: 'GOOGLE_APPLICATION_CREDENTIALS'
                    )

                ]) {


                    sh """


                    gcloud auth activate-service-account \
                    --key-file=$GOOGLE_APPLICATION_CREDENTIALS


                    gcloud config set project ${PROJECT_ID}



                    gcloud auth configure-docker \
                    ${REGION}-docker.pkg.dev \
                    --quiet


                    """

                }

            }

        }







        stage('Push Image To Artifact Registry') {


            steps {


                sh """


                docker push ${IMAGE_PATH}:${IMAGE_TAG}


                """

            }

        }







        stage('Deploy To VM') {


            steps {


                withCredentials([

                    sshUserPrivateKey(
                        credentialsId: "${SSH_CREDENTIAL}",
                        keyFileVariable: 'SSH_KEY'
                    )

                ]) {


                    sh """


                    chmod 600 $SSH_KEY



                    ssh \
                    -o StrictHostKeyChecking=no \
                    -i $SSH_KEY \
                    ${VM_USER}@${VM_IP} << EOF



                    echo "Configuring Docker authentication"



                    gcloud auth configure-docker \
                    ${REGION}-docker.pkg.dev \
                    --quiet




                    echo "Stopping old container"



                    docker stop ${CONTAINER_NAME} || true


                    docker rm ${CONTAINER_NAME} || true





                    echo "Pulling new image"



                    docker pull ${IMAGE_PATH}:${IMAGE_TAG}





                    echo "Starting container"



                    docker run -d \
                    --restart always \
                    -p 8000:8000 \
                    --name ${CONTAINER_NAME} \
                    ${IMAGE_PATH}:${IMAGE_TAG}



EOF


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
