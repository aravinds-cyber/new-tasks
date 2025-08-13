pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'echo "Building application..."'
                sh 'docker build -t myapp:${BRANCH_NAME} .'
                sh 'docker save myapp:${BRANCH_NAME} -o myapp_${BRANCH_NAME}.tar'
            }
        }

        stage('Deploy to Dev') {
            when { branch 'dev' }
            steps {
                sh """
                scp -o StrictHostKeyChecking=no myapp_dev.tar ubuntu@13.235.128.65:/tmp/
                ssh -o StrictHostKeyChecking=no ubuntu@13.235.128.65 '
                    docker load -i /tmp/myapp_dev.tar &&
                    docker stop myapp || true &&
                    docker rm myapp || true &&
                    docker run -d --name myapp -p 8080:8080 myapp:dev
                '
                """
            }
        }

        stage('Deploy to Staging') {
            when { branch 'staging' }
            steps {
                sh """
                scp -o StrictHostKeyChecking=no myapp_staging.tar ubuntu@15.206.72.230:/tmp/
                ssh -o StrictHostKeyChecking=no ubuntu@15.206.72.230 '
                    docker load -i /tmp/myapp_staging.tar &&
                    docker stop myapp || true &&
                    docker rm myapp || true &&
                    docker run -d --name myapp -p 8080:8080 myapp:staging
                '
                """
            }
        }
    }
}





