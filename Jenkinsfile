pipeline {
    agent any

    environment {
        APP_NAME = "myapp"
        DEV_SERVER = "13.235.128.65"
        STAGING_SERVER = "15.206.72.230"
        DOCKER_PORT = "8080"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image on Master') {
            steps {
                script {
                    echo "Building image for ${env.BRANCH_NAME}"
                    sh """
                        docker build -t ${APP_NAME}:${env.BRANCH_NAME} .
                        docker save ${APP_NAME}:${env.BRANCH_NAME} > ${APP_NAME}-${env.BRANCH_NAME}.tar
                    """
                }
            }
        }

        stage('Deploy to Target Server') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        echo "Deploying to DEV server"
                        sh """
                            scp ${APP_NAME}-${env.BRANCH_NAME}.tar ${DEV_SERVER}:/tmp/
                            ssh ${DEV_SERVER} '
                                docker load < /tmp/${APP_NAME}-${env.BRANCH_NAME}.tar &&
                                docker stop ${APP_NAME}-dev || true &&
                                docker rm ${APP_NAME}-dev || true &&
                                docker run -d --name ${APP_NAME}-dev -p 8081:${DOCKER_PORT} ${APP_NAME}:${env.BRANCH_NAME}
                            '
                        """
                    } else if (env.BRANCH_NAME == 'staging') {
                        echo "Deploying to STAGING server"
                        sh """
                            scp ${APP_NAME}-${env.BRANCH_NAME}.tar ${STAGING_SERVER}:/tmp/
                            ssh ${STAGING_SERVER} '
                                docker load < /tmp/${APP_NAME}-${env.BRANCH_NAME}.tar &&
                                docker stop ${APP_NAME}-staging || true &&
                                docker rm ${APP_NAME}-staging || true &&
                                docker run -d --name ${APP_NAME}-staging -p 8082:${DOCKER_PORT} ${APP_NAME}:${env.BRANCH_NAME}
                            '
                        """
                    } else {
                        echo "No deployment for branch ${env.BRANCH_NAME}"
                    }
                }
            }
        }
    }

    post {
        always {
            sh "rm -f ${APP_NAME}-${env.BRANCH_NAME}.tar || true"
            echo "Pipeline finished for branch: ${env.BRANCH_NAME}"
        }
    }
}




