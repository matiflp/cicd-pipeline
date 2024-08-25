pipeline {
    agent any
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
                        sh 'docker stop $(docker ps -q --filter ancestor=nodemain:v1.0)'
                        sh 'docker rm $(docker ps -a -q --filter ancestor=nodemain:v1.0)'
                        sh 'docker run -d --expose 3000 -p 3000:3000 nodemain:v1.0'
                    } else {
                        sh 'docker stop $(docker ps -q --filter ancestor=nodedev:v1.0)'
                        sh 'docker rm $(docker ps -a -q --filter ancestor=nodedev:v1.0)'
                        sh 'docker run -d --expose 3001 -p 3001:3001 nodedev:v1.0'
                    }
                }
            }
        }
    }
}
