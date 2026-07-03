pipeline {
    agent any
    
    tools {
        // This forces Jenkins to automatically install and expose the Docker CLI
        dockerTool 'my-docker' 
    }

    environment {
        IMAGE_NAME = "flask-test-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/chokkadevops/flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy App Container') {
            steps {
                sh '''
                    docker stop ${IMAGE_NAME} || true
                    docker rm ${IMAGE_NAME} || true
                    docker run -d -p 5000:5000 --name ${IMAGE_NAME} ${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo "Flask application built and containerized successfully!"
        }
        failure {
            echo "Pipeline failed. Check the console logs."
        }
    }
}