pipeline {
    agent any

    environment {
        IMAGE_VERSION = "1.0"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                script {
                    echo "Current Branch: ${env.BRANCH_NAME}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "Building for main branch"
                        sh 'docker build -t nodemain:${env.IMAGE_VERSION} .'
                    } else {
                        echo "Building for ${env.BRANCH_NAME} branch"
                        sh 'docker build -t nodedev:${env.IMAGE_VERSION} .'
                    }
                }
            }
        }

        stage('Stop and remove existing container') {
            steps {
                script {
                    // Define containername corresponding to branch
                    def containerName = (env.BRANCH_NAME == 'main') ? "nodemain" : "nodedev"
                    echo "Checking if container ${containerName} is running..."
                    // Check if container exist
                    def containerId = sh(script: "docker ps -q -f name=${containerName}", returnStdout: true).trim()
                    if (containerId) {
                       echo "Stopping and removing container ${containerName} with ID ${containerId}"
                       sh "docker stop ${containerName}"
                       sh "docker rm ${containerName}"
                    } else {
                        echo "No running container found with name ${containerName}"
                    }
                }
            }
        }



        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "Deploying main branch"
                        sh 'docker run -d --name nodemain --rm -p 3000:3000 nodemain:${env.IMAGE_VERSION}'
                    } else {
                        echo "Deploying ${env.BRANCH_NAME} branch"
                        sh 'docker run -d --name nodedev --rm -p 3001:3000 nodedev:${env.IMAGE_VERSION}'
                    }
                }
            }
        }
    }
}
