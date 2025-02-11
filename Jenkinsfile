pipeline {
    agent any

    environment {
        BRANCH_NAME = "${env.GIT_BRANCH}"
    }

    stages {
	
	stage('Set Environment Variables') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.APP_PORT = '3000'
                        env.DOCKER_IMAGE = 'nodemain:v1.0'
                    } else {
                        env.APP_PORT = '3001'
                        env.DOCKER_IMAGE = 'nodedev:v1.0'
                    }
                }
            }
        }


        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/kcsixr/module2t3.git'
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

        stage('Modify Logo & Ports') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'cp src/main.svg public/logo.svg'
                    } else {
                        sh 'cp src/dev.svg public/logo.svg'
                    }
                    sh "echo 'PORT=${APP_PORT}' > .env"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t myapp:${BRANCH_NAME} ."
            }
        }

        stage('Deploy') {
            steps {
                sh "docker run -d -p ${APP_PORT}:${APP_PORT} --name myapp_${BRANCH_NAME} myapp:${BRANCH_NAME}"
            }
        }
    }
}
