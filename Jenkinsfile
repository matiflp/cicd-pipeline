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
                        def vulnerabilities = sh(script: "
                                    trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress nodemain:v1.0
                                ", returnStdout: true).trim()
                                echo "Vulnerability Report for nodemain:v1.0:\n${vulnerabilities}"
                    } else {
                        def vulnerabilities = sh(script: "
                                    trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress nodedev:v1.0
                                ", returnStdout: true).trim()
                                echo "Vulnerability Report for nodedev:v1.0:\n${vulnerabilities}"
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin"
                            sh 'docker tag nodemain:v1.0  matilp95/nodemain:v1.0'
                            sh "docker push matilp95/nodemain:v1.0"
                        }
                    } else {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin"
                            sh 'docker tag nodedev:v1.0  matilp95/nodedev:v1.0'
                            sh "docker push matilp95/nodedev:v1.0"
                        }
                    }
                }
            }
        }
        // stage('Deploy') {
        //     steps {
        //         script {
        //             if (env.BRANCH_NAME == 'main') {
        //                 def containerID = sh(
        //                     script: "docker ps -q --filter 'publish=3000'",
        //                     returnStdout: true
        //                 ).trim()
                        
        //                 if (containerID) {
        //                     sh "docker stop ${containerID}"
        //                     sh "docker rm ${containerID}"
        //                 } else {
        //                     echo "No containers running with image nodemain:v1.0"
        //                 }
                        
        //                 sh "docker run -d --expose 3000 -p 3000:3000 nodemain:v1.0"                        
        //             } else {
        //                 def containerID = sh(
        //                     script: "docker ps -q --filter 'publish=3001'",
        //                     returnStdout: true
        //                 ).trim()
                        
        //                 if (containerID) {
        //                     sh "docker stop ${containerID}"
        //                     sh "docker rm ${containerID}"
        //                 } else {
        //                     echo "No containers running with image nodedev:v1.0"
        //                 }
                        
        //                 sh "docker run -d --expose 3001 -p 3001:3001 nodedev:v1.0"
        //             }
        //         }
        //     }
        // }
        stage('Lint Dockerfile with Hadolint') {
            steps {
                sh 'hadolint Dockerfile'
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
