@Library('PushDockerImageSharedLibrary') _
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
        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        def vulnerabilities = sh(script: "/usr/local/bin/trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress nodemain:v1.0", returnStdout: true).trim()
                        echo "Vulnerability Report for nodemain:v1.0:\n${vulnerabilities}"
                    } else {
                        def vulnerabilities = sh(script: "/usr/local/bin/trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress nodedev:v1.0", returnStdout: true).trim()
                        echo "Vulnerability Report for nodedev:v1.0:\n${vulnerabilities}"
                    }
                }
            }
        }
         stage('Push Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        pushDockerImage('nodemain', 'v1.0', 'docker-hub-credentials-id')
                    } else {
                        pushDockerImage('nodedev', 'v1.0', 'docker-hub-credentials-id')
                    }
                }
            }
        }
        stage('Lint Dockerfile with Hadolint') {
            steps {
                sh '/usr/local/bin/hadolint Dockerfile'
            }
        }
        stage('Trigger Deployment') {
            steps {
                script {
                    if (BRANCH_NAME == 'main') {
                        build job: 'Deploy_to_main'
                    } else if (BRANCH_NAME == 'dev') {
                        build job: 'Deploy_to_dev'
                    }
                }
            }
        }
    }
}
