pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir() // <-- Changed here
            }
        }

        stage('Clone repository') {
            steps {
                script {
                    checkout([$class: 'GitSCM',
                        branches: [[name: '*/main']],
                        extensions: [],
                        userRemoteConfigs: [[url: 'https://github.com/msystec/source-repo.git']]
                    ])
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    app = docker.build("jenkins_pipeline", "-f src/app/Dockerfile src/app")
                }
            }
        }

        stage('Docker Image push into ECR') {
            steps {
                script {
                    docker.withRegistry('https://267767410086.dkr.ecr.us-east-1.amazonaws.com/jenkins_pipeline', 'ecr:us-east-1:aws-credentials') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
    }
}
