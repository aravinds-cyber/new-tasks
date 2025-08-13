pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    dockerImage = docker.build("myapp:${imageTag}")
                }
            }
        }
        stage('Save and Transfer Docker Image') {
            steps {
                script {
                    def imageTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    def tarFile = "myapp_${imageTag}.tar"
                    sh "docker save -o ${tarFile} myapp:${imageTag}"

                    def server = ''
                    if (env.BRANCH_NAME == 'dev') {
                        server = 'dev-server-ip-or-hostname'
                    } else if (env.BRANCH_NAME == 'staging') {
                        server = 'staging-server-ip-or-hostname'
                    } else if (env.BRANCH_NAME == 'main') {
                        server = 'prod-server-ip-or-hostname'
                    } else {
                        error "No deployment target for branch ${env.BRANCH_NAME}"
                    }

                    sh "scp ${tarFile} user@${server}:/tmp/"
                }
            }
        }
        stage('Deploy on Target Server') {
            steps {
                script {
                    def imageTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    def tarFile = "myapp_${imageTag}.tar"
                    def server = ''
                    if (env.BRANCH_NAME == 'dev') {
                        server = '13.235.128.65'
                    } else if (env.BRANCH_NAME == 'staging') {
                        server = '15.206.72.230'
                    } else if (env.BRANCH_NAME == 'main') {
                        server = '3.111.147.44'
                    }

                    sh """
                    ssh user@${server} '
                    docker load -i /tmp/${tarFile} &&
                    docker stop myapp || true &&
                    docker rm myapp || true &&
                    docker run -d --name myapp -p 5000:5000 myapp:${imageTag}
                    '
                    """
                }
            }
        }
    }
}



