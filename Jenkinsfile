

// Developer : Chokkalingam
// Date : 04-07-2026
// Jenkinsfile is for Jenkins pipeline Build and Deployment. 
// We are using the IP defined by Docker Container to host the test project Flask inside the Docker container.


// pipeline {
//     agent any

//     environment {
//         APP_NAME = "flask-test-app"
//         PORT = "5000"
//         // Switched to Docker's dedicated internal Windows host gateway IP
//         DOCKER_API = "http://192.168.65.254:2375"

//         // Adding your new Docker tracking variables
//         IMAGE_NAME = "flask-test-app"
//         CONTAINER_NAME = "flask-test-app"
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/chokkadevops/flask-app.git'
//                 //echo "Code successfully fetched from GitHub repository."
//             }
//         }

//         /* Added by chokka. Not required to validate the folder paths.
//         stage('Validate Project Assets') {
//             steps {
//                 // Added by Chokka - To verify the list of files in the project workspace.
//                 // This prints the files to the Jenkins log, but returns nothing to the pipeline memory
//                 sh 'ls -la'
//             }
//         }*/

//         stage('Build Image via Docker API') {
//             steps {
//                 //echo "Packaging workspace and sending to Windows Docker API..."
//                 sh '''
//                     # Tar the current directory safely excluding the tarball itself
//                     tar --exclude='workspace.tar' -cvf workspace.tar .
                    
//                     # Submit the tarball directly to Docker Desktop's build engine API
//                     curl -X POST -H "Content-Type: application/x-tar" \
//                       --data-binary @workspace.tar \
//                       "${DOCKER_API}/build?t=${APP_NAME}:latest"
//                 '''
//             }
//         }

//         stage('Deploy Container via Docker API') {
//             steps {
//                 echo "Managing container lifestyle via REST API..."
//                 script {
//                     // 1. Stop and delete old container if it exists
//                     sh """
//                         curl -X POST "${DOCKER_API}/containers/${APP_NAME}/stop" || true
//                         curl -X DELETE "${DOCKER_API}/containers/${APP_NAME}" || true
//                     """

//                     // 2. Create the new container instance configuration via JSON payload
//                     sh """
//                         curl -X POST -H "Content-Type: application/json" \
//                           -d '{
//                                 "Image": "${APP_NAME}:latest",
//                                 "ExposedPorts": { "${PORT}/tcp": {} },
//                                 "HostConfig": {
//                                     "PortBindings": { "${PORT}/tcp": [{ "HostPort": "${PORT}" }] }
//                                 }
//                               }' \
//                           "${DOCKER_API}/containers/create?name=${APP_NAME}"
//                     """

//                     // 3. Start the newly created container
//                     sh """
//                         curl -X POST "${DOCKER_API}/containers/${APP_NAME}/start"
//                     """
//                 }
//             }
//         }
//     }

      
//    post {
//         always {
//             echo "Pipeline build process completed."
//             // Cleaned up the failing permission commands
//         }

//         success {
//             echo "Flask application successfully built and deployed via Windows Docker API!"
//         }

//         failure {
//             echo "Pipeline execution failed."
//         }
//     }

   
// }


// Added by Chokka for choosing branch : main, dev01, dev02

pipeline {
    agent any

    environment {
        // Automatically appends the branch name to prevent overwriting containers/images
        // e.g., flask-test-app-main, flask-test-app-dev01
        APP_NAME        = "flask-test-app-${env.BRANCH_NAME.toLowerCase()}"
        CONTAINER_NAME  = "flask-test-app-${env.BRANCH_NAME.toLowerCase()}"
        
        // Dynamic image tagging based on branch name instead of just 'latest'
        IMAGE_TAG       = "${env.BRANCH_NAME.toLowerCase()}"

        // Switched to Docker's dedicated internal Windows host gateway IP
        DOCKER_API      = "http://192.168.65.254:2375"

        // Dynamic Port Routing: Assigns a unique port based on the branch so they don't collide
        PORT = "${env.BRANCH_NAME == 'main' ? '5000' : env.BRANCH_NAME == 'dev01' ? '5001' : '5002'}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // CRITICAL FIX: Automatically checks out whichever branch triggered the pipeline
                checkout scm
                echo "Successfully fetched code from branch: ${env.BRANCH_NAME}"
            }
        }

        stage('Build Image via Docker API') {
            steps {
                sh '''
                    # Tar the current directory safely excluding the tarball itself
                    tar --exclude='workspace.tar' -cvf workspace.tar .
                    
                    # Submit the tarball directly to Docker Desktop's build engine API
                    curl -X POST -H "Content-Type: application/x-tar" \
                      --data-binary @workspace.tar \
                      "${DOCKER_API}/build?t=${APP_NAME}:${IMAGE_TAG}"
                '''
            }
        }

        stage('Deploy Container via Docker API') {
            steps {
                echo "Managing container lifestyle for ${APP_NAME} on port ${PORT}..."
                script {
                    // 1. Stop and delete old branch container if it exists
                    sh """
                        curl -X POST "${DOCKER_API}/containers/${CONTAINER_NAME}/stop" || true
                        curl -X DELETE "${DOCKER_API}/containers/${CONTAINER_NAME}" || true
                    """

                    // 2. Create the new container instance configuration via JSON payload
                    sh """
                        curl -X POST -H "Content-Type: application/json" \
                          -d '{
                                "Image": "${APP_NAME}:${IMAGE_TAG}",
                                "ExposedPorts": { "5000/tcp": {} },
                                "HostConfig": {
                                    "PortBindings": { "5000/tcp": [{ "HostPort": "${PORT}" }] }
                                }
                              }' \
                          "${DOCKER_API}/containers/create?name=${CONTAINER_NAME}"
                    """

                    // 3. Start the newly created container
                    sh """
                        curl -X POST "${DOCKER_API}/containers/${CONTAINER_NAME}/start"
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline build process completed for branch: ${env.BRANCH_NAME}."
        }

        success {
            echo "Flask application successfully built and deployed to port ${PORT}!"
        }

        failure {
            echo "Pipeline execution failed for branch: ${env.BRANCH_NAME}."
        }
    }
}