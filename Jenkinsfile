pipeline {
    agent any

    environment {
        APP_NAME = "myapp"
        DEV_SERVER = "ubuntu@13.235.128.65"
        STAGING_SERVER = "ubuntu@15.206.72.230"
        SSH_KEY = "/root/.ssh/my-key.pem"  // path to your EC2 private key in Jenkins container
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Build image with branch-specific tag
                    def tag = "${env.BRANCH_NAME}-1"
                    sh """
                        docker build -t ${APP_NAME}:${tag} .
                        docker save ${APP_NAME}:${tag} -o ${APP_NAME}_${tag}.tar
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            when { branch 'dev' }
            steps {
                script {
                    def tag = "dev-1"
                    sh """
                        scp -i ${SSH_KEY} ${APP_NAME}_${tag}.tar ${DEV_SERVER}:/tmp/
                        ssh -i ${SSH_KEY} ${DEV_SERVER} '
                            docker load -i /tmp/${APP_NAME}_${tag}.tar &&
                            docker rm -f ${APP_NAME} || true &&
                            docker run -d --name ${APP_NAME} -p 3000:3000 ${APP_NAME}:${tag}
                        '
                    """
                }
            }
        }

        stage('Deploy to Staging') {
            when { branch 'staging' }
            steps {
                script {
                    def tag = "staging-1"
                    sh """
                        scp -i ${SSH_KEY} ${APP_NAME}_${tag}.tar ${STAGING_SERVER}:/tmp/
                        ssh -i ${SSH_KEY} ${STAGING_SERVER} '
                            docker load -i /tmp/${APP_NAME}_${tag}.tar &&
                            docker rm -f ${APP_NAME} || true &&
                            docker run -d --name ${APP_NAME} -p 3000:3000 ${APP_NAME}:${tag}
                        '
                    """
                }
            }
        }
    }
}






