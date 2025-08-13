pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
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
                    
                    // Save Docker image as tar
                    sh "docker save -o ${tarFile} myapp:${imageTag}"
                    
                    // Copy tar to target server based on branch
                    if (env.BRANCH_NAME == 'dev') {
                        sh "scp ${tarFile} user@dev-server:/tmp/"
                    } else if (env.BRANCH_NAME == 'staging') {
                        sh "scp ${tarFile} user@staging-server:/tmp/"
                    } else if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'prod') {
                        sh "scp ${tarFile} user@prod-server:/tmp/"
                    } else {
                        error "No deployment target for branch ${env.BRANCH_NAME}"
                    }
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
                        server = 'dev-server'
                    } else if (env.BRANCH_NAME == 'staging') {
                        server = 'staging-server'
                    } else if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'prod') {
                        server = 'prod-server'
                    } else {
                        error "No deployment target for branch ${env.BRANCH_NAME}"
                    }
                    
                    // SSH into server, load image, stop old container, run new container
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
    
    post {
        always {
            echo "Done deploying branch ${env.BRANCH_NAME}"
        }
    }
}



