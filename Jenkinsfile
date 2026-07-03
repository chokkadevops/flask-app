pipeline {
    agent any

    environment {
        APP_NAME = "flask-test-app"
        PORT = "5000"
        // Explicitly enforces TCP routing for all Docker CLI commands in this pipeline
        DOCKER_HOST = "tcp://host.docker.internal:2375"
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

        stage('Build Docker Image') {
            steps {
                echo "Building application Docker image using TCP daemon connection..."
                // Builds the image directly on your Windows Docker Desktop engine
                sh "docker build -t ${APP_NAME}:latest ."
            }
        }

        stage('Deploy Application Container') {
            steps {
                echo "Deploying container to Windows host environment..."
                script {
                    // Stop and remove old container instances safely if present
                    sh '''
                    if docker ps -a --format '{{.Names}}' | grep -q "^${APP_NAME}$"; then
                        echo "Found existing container. Removing..."
                        docker rm -f ${APP_NAME} || true
                    fi
                    '''
                    // Spin up the freshly built image
                    sh "docker run -d -p ${PORT}:${PORT} --name ${APP_NAME} ${APP_NAME}:latest"
                }
            }
        }
    }

    post {
        success {
            echo "Flask application pipeline executed and deployed perfectly!"
        }
        failure {
            echo "Pipeline encountered an error during execution."
        }
    }
}