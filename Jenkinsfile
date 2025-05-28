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

        stage('Junit Test & Security Tests') {
            steps {
                echo 'Empty'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    app = docker.build("jenkins_pipeline_1", "-f src/app/Dockerfile src/app")
                }
            }
        }

        // ✅ Moved inside stages block
        stage('Docker Image Push into ECR') {
            steps {
                script {
                    docker.withRegistry(
                        'https://017820667794.dkr.ecr.us-east-1.amazonaws.com/jenkins_pipeline_1',
                        'ecr:us-east-1:aws-ecr'
                    ) {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
    }  // ✅ Properly closes the stages block
}
