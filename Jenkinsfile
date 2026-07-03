pipeline {
    agent any

    environment {
        APP_NAME = "flask-test-app"
        PORT = "5000"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/chokkadevops/flask-app.git'
                echo "Code successfully fetched from GitHub repository."
            }
        }

        stage('Validate Project Assets') {
            steps {
                sh '''
                    echo "Checking structural components..."
                    ls -la
                '''
            }
        }

        stage('Build & Run App Container') {
            agent {
                docker {
                    image 'python:3.10-slim'
                    args "-p ${PORT}:${PORT}"
                }
            }
            steps {
                echo "Jenkins plugin has provisioned the Python environment seamlessly."
                sh 'pip install -r requirements.txt'
                echo "Starting the Flask web application..."
                sh 'python app.py &'
                sleep 5
            }
        }
    }

    post {
        success {
            echo "Flask application pipeline executed perfectly!"
        }
        failure {
            echo "Pipeline encountered an error during the run phase."
        }
    }
}