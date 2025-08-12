pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Map branch to image tag
                    IMAGE_TAG = (BRANCH_NAME == "main" || BRANCH_NAME == "prod") ? "prod" : BRANCH_NAME
                    
                    echo "Building Docker image for branch: ${BRANCH_NAME} → tag: ${IMAGE_TAG}"
                    sh "docker build -t myapp:${IMAGE_TAG} ."
                }
            }
        }

        stage('Deploy Locally') {
            steps {
                script {
                    // Map branch to image tag again for safety
                    IMAGE_TAG = (BRANCH_NAME == "main" || BRANCH_NAME == "prod") ? "prod" : BRANCH_NAME

                    // Remove any existing container with same name
                    sh "docker ps -aq --filter name=myapp-${IMAGE_TAG} | xargs -r docker rm -f"

                    if (BRANCH_NAME == 'dev') {
                        echo "Running Dev environment..."
                        sh "docker run -d -p 5001:5000 --name myapp-dev myapp:${IMAGE_TAG}"
                    } else if (BRANCH_NAME == 'staging') {
                        echo "Running Staging environment..."
                        sh "docker run -d -p 5002:5000 --name myapp-staging myapp:${IMAGE_TAG}"
                    } else if (BRANCH_NAME == 'main' || BRANCH_NAME == 'prod') {
                        echo "Running Production environment..."
                        sh "docker run -d -p 5003:5000 --name myapp-prod myapp:${IMAGE_TAG}"
                    } else {
                        echo "No deployment rules for this branch."
                    }
                }
            }
        }
    }
}


