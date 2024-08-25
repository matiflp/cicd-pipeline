pipeline {
    agent any
    tools {
        nodejs 'NodeJS 7.8.0'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
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
                        sh 'docker build -t nodemain:v1.0 .'
                    } else {
                        sh 'docker build -t nodedev:v1.0 .'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        def containerID = sh(
                            script: "docker ps -q --filter 'publish=3000'",
                            returnStdout: true
                        ).trim()
                        
                        if (containerID) {
                            sh "docker stop ${containerID}"
                            sh "docker rm ${containerID}"
                        } else {
                            echo "No containers running with image nodemain:v1.0"
                        }
                        
                        sh "docker run -d --expose 3000 -p 3000:3000 nodemain:v1.0"                        
                    } else {
                        def containerID = sh(
                            script: "docker ps -q --filter 'publish=3001'",
                            returnStdout: true
                        ).trim()
                        
                        if (containerID) {
                            sh "docker stop ${containerID}"
                            sh "docker rm ${containerID}"
                        } else {
                            echo "No containers running with image nodedev:v1.0"
                        }
                        
                        sh "docker run -d --expose 3001 -p 3001:3001 nodedev:v1.0"
                    }
                }
            }
        }
    }
}
