

// Developer : Chokkalingam
// Date : 04-07-2026
// Jenkinsfile is for Jenkins pipeline Build and Deployment. 
// We are using the IP defined by Docker Container to host the test project Flask inside the Docker container.



pipeline {
    agent any

    environment {
        APP_NAME = "flask-test-app"
        PORT = "5000"
        // Switched to Docker's dedicated internal Windows host gateway IP
        DOCKER_API = "http://192.168.65.254:2375"

        // Adding your new Docker tracking variables
        IMAGE_NAME = "flask-test-app"
        CONTAINER_NAME = "flask-test-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/chokkadevops/flask-app.git'
                //echo "Code successfully fetched from GitHub repository."
            }
        }

        stage('Validate Project Assets') {
            steps {
                sh 'ls -la'
            }
        }

        stage('Build Image via Docker API') {
            steps {
                //echo "Packaging workspace and sending to Windows Docker API..."
                sh '''
                    # Tar the current directory safely excluding the tarball itself
                    tar --exclude='workspace.tar' -cvf workspace.tar .
                    
                    # Submit the tarball directly to Docker Desktop's build engine API
                    curl -X POST -H "Content-Type: application/x-tar" \
                      --data-binary @workspace.tar \
                      "${DOCKER_API}/build?t=${APP_NAME}:latest"
                '''
            }
        }

        stage('Deploy Container via Docker API') {
            steps {
                echo "Managing container lifestyle via REST API..."
                script {
                    // 1. Stop and delete old container if it exists
                    sh """
                        curl -X POST "${DOCKER_API}/containers/${APP_NAME}/stop" || true
                        curl -X DELETE "${DOCKER_API}/containers/${APP_NAME}" || true
                    """

                    // 2. Create the new container instance configuration via JSON payload
                    sh """
                        curl -X POST -H "Content-Type: application/json" \
                          -d '{
                                "Image": "${APP_NAME}:latest",
                                "ExposedPorts": { "${PORT}/tcp": {} },
                                "HostConfig": {
                                    "PortBindings": { "${PORT}/tcp": [{ "HostPort": "${PORT}" }] }
                                }
                              }' \
                          "${DOCKER_API}/containers/create?name=${APP_NAME}"
                    """

                    // 3. Start the newly created container
                    sh """
                        curl -X POST "${DOCKER_API}/containers/${APP_NAME}/start"
                    """
                }
            }
        }
    }

    
   // Chokka - Adding log entry to monitor the application running inside the container.

   post {
        always {
            echo "Fetching logs via API and generating file..."
            
            // The script block allows variable declarations inside declarative pipelines
            script {
                def logEntries = sh(
                    script: "curl -s \"${env.DOCKER_API}/containers/${env.CONTAINER_NAME}/logs?stdout=true&stderr=true&tail=10&timestamps=true\"", 
                    returnStdout: true
                ).trim()
                
                // Writes the file cleanly into the workspace directory
                writeFile file: "flask_logoutput.txt", text: logEntries
            }
        }

        success {
            echo "Flask application successfully built and deployed via Windows Docker API!"
        }

        failure {
            echo "Pipeline execution failed."
        }
    }

   
}