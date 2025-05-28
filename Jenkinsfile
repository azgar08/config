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
                        userRemoteConfigs: [[url: 'https://https://github.com/azgar08/config.git']]
                    ])
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    app = docker.build("jenkins_pipeline_1", "-f src/app/Dockerfile src/app")
                }
            }
        }
    }  // Close stages block
}  // Close pipeline block
