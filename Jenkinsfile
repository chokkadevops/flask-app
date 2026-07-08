

// Developer : Chokkalingam
// Date : 04-07-2026
// Jenkinsfile is for Jenkins pipeline Build and Deployment. 
// We are using the IP defined by Docker Container to host the test project Flask inside the Docker container.


// pipeline {
//      agent any

//      environment {
//          APP_NAME = "flask-test-app"
//          PORT = "5000"
//          // Switched to Docker's dedicated internal Windows host gateway IP
//         DOCKER_API = "http://192.168.65.254:2375"

//          // Adding your new Docker tracking variables
//          IMAGE_NAME = "flask-test-app"
//          CONTAINER_NAME = "flask-test-app"
//      }

//      stages {
//          stage('Checkout Code') {
//              steps {
//                  git branch: 'main', url: 'https://github.com/chokkadevops/flask-app.git'
//                  //echo "Code successfully fetched from GitHub repository."
//              }
//          }

//          /* Added by chokka. Not required to validate the folder paths.
//          stage('Validate Project Assets') {
//              steps {
//                  // Added by Chokka - To verify the list of files in the project workspace.
//                  // This prints the files to the Jenkins log, but returns nothing to the pipeline memory
//                  sh 'ls -la'
//              }
//          }*/

         
//          // tar -- workspace.tar to zip all the files for deployment
//          // create verbose file
//          //Post : Jenkins to send data to the Docker server
//          // Attaches actual project to data binary workspace tar
//          // Container image is created via Docker API
//          stage('Build Image via Docker API') {
//              steps {
//                  //echo "Packaging workspace and sending to Windows Docker API..."
//                  sh '''
//                      # Tar the current directory safely excluding the tarball itself
//                      tar --exclude='workspace.tar' -cvf workspace.tar .
                    
//                      # Submit the tarball directly to Docker Desktop's build engine API
//                      curl -X POST -H "Content-Type: application/x-tar" \
//                        --data-binary @workspace.tar \
//                        "${DOCKER_API}/build?t=${APP_NAME}:latest"
//                  '''
//              }
//          }

//          // POST DELETE = stop and delete the old container ( clean up )
//          // step : 2 container assemble all necessary steps
//          // writable file layer - log, container network

//          stage('Deploy Container via Docker API') {
//              steps {
//                  echo "Managing container lifestyle via REST API..."
//                  script {
//                      // 1. Stop and delete old container if it exists
//                      sh """
//                          curl -X POST "${DOCKER_API}/containers/${APP_NAME}/stop" || true
//                          curl -X DELETE "${DOCKER_API}/containers/${APP_NAME}" || true
//                      """

//                      // 2. Create the new container instance configuration via JSON payload
//                      sh """
//                          curl -X POST -H "Content-Type: application/json" \
//                            -d '{
//                                  "Image": "${APP_NAME}:latest",
//                                  "ExposedPorts": { "${PORT}/tcp": {} },
//                                  "HostConfig": {
//                                      "PortBindings": { "${PORT}/tcp": [{ "HostPort": "${PORT}" }] }
//                                  }
//                                }' \
//                            "${DOCKER_API}/containers/create?name=${APP_NAME}"
//                      """

//                      // 3. Start the newly created container
//                      sh """
//                          curl -X POST "${DOCKER_API}/containers/${APP_NAME}/start"
//                      """
//                  }
//              }
//          }
//      }

      
//     post {
//          always {
//              echo "Pipeline build process completed."
//              // Cleaned up the failing permission commands
//          }

//          success {
//              echo "Flask application successfully built and deployed via Windows Docker API!"
//          }

//          failure {
//              echo "Pipeline execution failed."
//          }
//      }

   
//  }




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
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        // Docker 

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

    post {
        success {
            echo "CI/CD Pipeline executed successfully. App live at http://localhost:${HOST_PORT}"
        }
        failure {
            echo "Pipeline failed. Check stage logs for details."
        }
    }
}