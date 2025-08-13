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

        stage('Build Docker Image Locally') {
            steps {
                script {
                    echo "Building image for ${env.BRANCH_NAME}"
                    sh """
                        docker build -t ${APP_NAME}:${env.BRANCH_NAME} .
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
                            scp -r . ${DEV_SERVER}:/tmp/${APP_NAME}
                            ssh ${DEV_SERVER} '
                                cd /tmp/${APP_NAME} &&
                                docker stop ${APP_NAME}-dev || true &&
                                docker rm ${APP_NAME}-dev || true &&
                                docker build -t ${APP_NAME}:dev . &&
                                docker run -d --name ${APP_NAME}-dev -p 8081:${DOCKER_PORT} ${APP_NAME}:dev
                            '
                        """
                    } else if (env.BRANCH_NAME == 'staging') {
                        echo "Deploying to STAGING server"
                        sh """
                            scp -r . ${STAGING_SERVER}:/tmp/${APP_NAME}
                            ssh ${STAGING_SERVER} '
                                cd /tmp/${APP_NAME} &&
                                docker stop ${APP_NAME}-staging || true &&
                                docker rm ${APP_NAME}-staging || true &&
                                docker build -t ${APP_NAME}:staging . &&
                                docker run -d --name ${APP_NAME}-staging -p 8082:${DOCKER_PORT} ${APP_NAME}:staging
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
            echo "Pipeline finished for branch: ${env.BRANCH_NAME}"
        }
    }
}

}



