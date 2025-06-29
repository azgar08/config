pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '017820667794.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_NAME = 'jenkins_pipeline_1'
        HELM_GIT_REPO_URL = 'https://github.com/azgar08/config.git'
        GIT_REPO_EMAIL = 'azgarali.a.n@gmail.com'
        GIT_REPO_BRANCH = 'main'
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
                    def app = docker.build("${ECR_REGISTRY}/${IMAGE_NAME}", "-f src/app/Dockerfile src/app")
                    env.IMAGE = app.id
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
                    def imageName = "${ECR_REGISTRY}/${IMAGE_NAME}"
                    sh "docker tag $IMAGE $imageName:${BUILD_NUMBER}"
                    sh "docker tag $IMAGE $imageName:latest"
                    sh "docker push $imageName:${BUILD_NUMBER}"
                    sh "docker push $imageName:latest"
                }
            }
        }

        stage('Updating HELM Charts') {
            steps {
                cleanWs()
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${GIT_REPO_BRANCH}"]],
                        userRemoteConfigs: [[
                            url: "${HELM_GIT_REPO_URL}",
                            credentialsId: 'github-credentials'
                        ]]
                    ])
                }

                withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    script {
                        def tag = "${env.BUILD_NUMBER}"
                        sh """
                            git config user.name "$GIT_USER"
                            git config user.email "${GIT_REPO_EMAIL}"

                            git checkout ${GIT_REPO_BRANCH} || git checkout -b ${GIT_REPO_BRANCH}
                            yq eval '.deployment.container.image.tag = "${tag}"' -i deploy/helm/hello-kubernetes/values.yaml

                            git add deploy/helm/hello-kubernetes/values.yaml
                            git commit -m "Triggered Build - update tag to ${tag}" || echo "No changes to commit"
                            git push https://${GIT_USER}:${GIT_TOKEN}@github.com/azgar08/config.git ${GIT_REPO_BRANCH}
                        """
                    }
                    sleep(time: 3, unit: "MINUTES")
                }
            }
        }

        stage('Test the Latest Application Tag') {
            steps {
                script {
                    def valuesFile = 'deploy/helm/hello-kubernetes/values.yaml'
                    def expectedTag = "${env.BUILD_NUMBER}".trim()

                    def currentTag = sh(
                        script: "yq eval '.deployment.container.image.tag' ${valuesFile}",
                        returnStdout: true
                    ).trim()

                    echo "✅ Expected tag: ${expectedTag}"
                    echo "📄 Found tag in values.yaml: ${currentTag}"

                    if (currentTag == expectedTag) {
                        echo "Application is updated !!!"
                    } else {
                        echo "Application is not updated with the latest tag"
                    }
                }
            }
        }
    }
}
