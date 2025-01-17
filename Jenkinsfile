pipeline {
    agent any

    environment { 
        NETLIFY_SITE_ID = 'c57261eb-c8bf-40f5-9297-aacc858aee8b'
    }

    stages {
                stage("Deploy prod"){
            agent {
                docker {
                    image "amazon/aws-cli"
                    args "--entrypoint=''"
                    reuseNode true
                }
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'd378a78c-6579-4117-b8ba-a6ff9e91cb3a', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh """
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                    """
                }

            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
    }
}
