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
                    echo "Verifying application file content..."
                    cat app.py | head -n 5
                '''
            }
        }

        stage('Build & Run App Container') {
            agent {
                docker {
                   //  image 'python:3.10-slim'
                    // // This dynamically maps port 5000 from your host machine to the container
                    // args "-p ${PORT}:${PORT}"

                    // The plugin handles building the Dockerfile automatically
                    def appImage = docker.build("chokkadevops/flask-app:latest")
            
                    // The plugin handles running the container on port 5000
                    appImage.run("-p 5000:5000 --name running-flask-app")

                }
            }
            steps {
                echo "Jenkins plugin has provisioned the Python environment seamlessly."
                
                // Install the dependencies inside the managed container
                sh 'pip install -r requirements.txt'
                
                echo "Starting the Flask web application..."
                // Starts the application in the background
                sh 'python app.py &'
                
                // Give it a moment to initialize
                sleep 5
            }
        }
    }

    post {
        success {
            echo "Flask application pipeline simulation executed perfectly!"
        }
        failure {
            echo "Pipeline encountered an error during the run phase."
        }
    }
}