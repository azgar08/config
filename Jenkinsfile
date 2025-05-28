pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '017820667794.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_NAME = 'jenkins_pipeline_1'
    }

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

        stage('Junit Test & Security Tests') {
            steps {
                echo 'Running Junit and Security Tests... (placeholder)'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    app = docker.build("${ECR_REGISTRY}/${IMAGE_NAME}", "-f src/app/Dockerfile src/app")
                }
            }
        }

        stage('Authenticate to ECR') {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Docker Image Push into ECR') {
            steps {
                script {
                    app.push("${BUILD_NUMBER}")
                    app.push("latest")
                }
            }
        }
    }
}
