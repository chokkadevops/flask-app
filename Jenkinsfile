

// Developer : Chokkalingam
// Date : 08-07-2026
// Jenkinsfile is for Jenkins pipeline Build and Deployment. 
// We are using the IP defined by Docker Container to host the test project Flask inside the Docker container.


pipeline {
    agent any

    environment {
        IMAGE_NAME = "flask-test-app"
        CONTAINER_NAME = "flask-test-app"
        HOST_PORT = "5000" 
    }

    // git branch: 'main', url: https://github.com/chokkadevops/flask-app.git
    // choose specific branch - main or dev. pulls the code from exact branch as  initiated.
    // safer as compared to hardcoding the git repo link

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        // Docker image is created. 

        stage('Docker Build (CI)') {
            steps {
                echo "Starting Docker build process..."
                // This triggers the internal 'npm run build' inside the multi-stage Dockerfile
                // -t tag to give a valid name for docker image file instead of assigned id.
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        // Docker assemble files layer by layer to the image.

        stage('Docker Deploy (CD)') {
            steps {
                echo "Deploying built image to runtime container..."
                script {
                    // Gracefully handle removing existing container if it exists
                    def status = sh(script: "docker ps -a -q -f name=${CONTAINER_NAME}", returnStdout: true).trim()
                    if (status) {
                        echo "Existing container found. Stopping and removing..."
                        sh "docker rm -f ${CONTAINER_NAME}"
                    }
                }
                // Run the newly built container mapping your local port to Nginx port 80
                // Change the -p mapping from :80 to :${HOST_PORT}
                sh "docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:${HOST_PORT} ${IMAGE_NAME}:latest"
            }
        }
    }

    
}