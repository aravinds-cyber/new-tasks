pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image for branch: ${BRANCH_NAME}"
                    sh "docker build -t myapp:${BRANCH_NAME} ."
                }
            }
        }

        stage('Deploy Locally') {
            steps {
                script {
                    // Stop any existing container for this branch
                    sh "docker ps -aq --filter name=myapp-${BRANCH_NAME} | xargs -r docker rm -f"

                    if (BRANCH_NAME == 'dev') {
                        echo "Running Dev environment..."
                        sh "docker run -d -p 5001:5000 --name myapp-dev myapp:dev"
                    } else if (BRANCH_NAME == 'staging') {
                        echo 'Running Staging environment...'
                        sh "docker run -d -p 5002:5000 --name myapp-staging myapp:staging"
                    } else if (BRANCH_NAME == 'main' || BRANCH_NAME == 'prod') {
                        echo 'Running Production environment...'
                        sh "docker run -d -p 5003:5000 --name myapp-prod myapp:prod"
                    } else {
                        echo "No deployment rules for this branch."
                    }
                }
            }
        }
    }
}

