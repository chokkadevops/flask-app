pipeline {
    agent any

    environment {
        // Define reusable variables
        IMAGE_NAME    = 'chokkadevops/flask-app'
        IMAGE_TAG     = 'latest'
        CONTAINER_NAME = 'running-flask-app'
        APP_PORT       = '5000'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Fetching the latest source code from GitHub...'
                // Code is automatically checked out by Jenkins if using "Pipeline from SCM"
                sh 'ls -la'
            }
        }

        stage('Validate Project Assets') {
            steps {
                echo 'Checking structural components...'
                sh '''
                    if [ ! -f app.py ]; then
                        echo "ERROR: app.py missing!"
                        exit 1
                    fi
                    if [ ! -f Dockerfile ]; then
                        echo "ERROR: Dockerfile missing!"
                        exit 1
                    fi
                '''
            }
        }

        stage('Build & Run App Container') {
            steps {
                echo 'Stopping and removing existing application container if running...'
                // || true prevents the pipeline from failing if the container doesn't exist yet
                sh "docker rm -f ${CONTAINER_NAME} || true"

                echo "Building Docker Image: ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."

                echo "Deploying Container to http://localhost:${APP_PORT}..."
                sh "docker run -d -p ${APP_PORT}:${APP_PORT} --name ${CONTAINER_NAME} ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
    }

    post {
        always {
            echo 'Pipeline build and deployment process completed.'
        }
        success {
            echo "SUCCESS: Application is running successfully at http://localhost:${APP_PORT}"
        }
        failure {
            echo "FAILURE: The build or deployment stage encountered an error."
            echo "Checking system docker container status logs..."
            // Tries to grab the crash logs if the container failed to initialize properly
            sh "docker logs ${CONTAINER_NAME} --tail 20 || true"
        }
    }
}