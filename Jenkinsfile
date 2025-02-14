pipeline {
    agent any

    environment {
        IMAGE_VERSION = "1.0"
	DOCKER_HUB_REPO = "kcsixr"
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

       stage('Lint Dockerfile') {
            steps {
                script {
                    echo "Linting Dockerfile with Hadolint"
                    sh "hadolint Dockerfile"
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
                        sh "docker build -t ${env.DOCKER_HUB_REPO}/nodemain:${env.IMAGE_VERSION} ."
                    } else {
                        echo "Building for ${env.BRANCH_NAME} branch"
                        sh "'docker build -t ${env.DOCKER_HUB_REPO}/nodedev:${env.IMAGE_VERSION} ."
                    }
                }
            }
        }

        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    def imageName = (env.BRANCH_NAME == 'main') ? "${env.DOCKER_HUB_REPO}/nodemain:${env.IMAGE_VERSION}" : "${env.DOCKER_HUB_REPO}/nodedev:${env.IMAGE_VERSION}"
                    echo "Scanning Docker image: ${imageName}"
                    def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${imageName}", returnStdout: true).trim()
                    echo "Vulnerability Report:\n${vulnerabilities}"
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
                       sh "docker stop ${containerId}"
                    } else {
                        echo "No running container found with name ${containerName}"
                    }
                }
            }
        }


        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                        // Используем переменные, созданные withCredentials
                        sh '''
                            echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USER --password-stdin
                        '''
                        if (env.BRANCH_NAME == 'main') {
                            sh "docker push ${env.DOCKER_HUB_REPO}/nodemain:${env.IMAGE_VERSION}"
                        } else {
                            sh "docker push ${env.DOCKER_HUB_REPO}/nodedev:${env.IMAGE_VERSION}"
                        }
                        sh "docker logout"
                    }
                }
            }
        }

// Deploy before
//        stage('Deploy') {
//            steps {
//                script {
//                    if (env.BRANCH_NAME == 'main') {
//                        echo "Deploying main branch"
//                        sh "docker run -d --name nodemain --rm -p 3000:3000 nodemain:${env.IMAGE_VERSION}"
//                    } else {
//                        echo "Deploying ${env.BRANCH_NAME} branch"
//                        sh "docker run -d --name nodedev --rm -p 3001:3000 nodedev:${env.IMAGE_VERSION}"
//                    }
//                }
//            }
//        }


        stage('Trigger Deploy Pipeline') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        build(job: 'Deploy_to_main', wait: false)
                    } else {
                        build(job: 'Deploy_to_dev', wait: false)
                    }
                }
            }
        }


    }
}



