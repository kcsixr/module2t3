pipeline {
    agent any

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
                        sh 'docker build -t nodemain:v1.0 .'
                    } else {
                        echo "Building for ${env.BRANCH_NAME} branch"
                        sh 'docker build -t nodedev:v1.0 .'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "Deploying main branch"
                        sh 'docker run -d --name nodemain --rm -p 3000:3000 nodemain:v1.0'
                    } else {
                        echo "Deploying ${env.BRANCH_NAME} branch"
                        sh 'docker run -d --name nodedev --rm -p 3001:3000 nodedev:v1.0'
                    }
                }
            }
        }
    }
}
