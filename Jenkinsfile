pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone repository') {
            steps {
                script {
                    checkout([$class: 'GitSCM',
                        branches: [[name: '*/main']],
                        extensions: [],
                        userRemoteConfigs: [[url: 'https://github.com/azgar08/config.git']]
                    ])
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Build Docker image from src/app directory
                    app = docker.build("jenkins_pipeline_1", "-f src/app/Dockerfile src/app")
                }
            }
        }

        stage('Docker Image push into ECR') {
            steps {
                script {
                    docker.withRegistry('https://267767410086.dkr.ecr.us-east-1.amazonaws.com/jenkins_pipeline_1', 'ecr:us-east-1:aws-credentials') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
    }
}
