pipeline {
    agent any

    environment { 
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = "myjenkinsapp"
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_ECS_CLUSTER = "LearnJenkinsApp-Cluster-prod"
        AWS_ECS_CLUSTER_SERVICE_PROD = "LearnJenkinApp-Service-Prod"
        AWS_ECS_TD_PROD = "LearnJenkinsApp-TaskDeifinition-Prod"

    }

    stages {
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

        stage('Build Docker'){
            agent {
                docker {
                    image "my-aws-cli"
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                sh "docker build -t $APP_NAME:$REACT_APP_VERSION ."
            }
        }

        stage("Deploy prod"){
            agent {
                docker {
                    image "my-aws-cli"
                    args "--entrypoint=''"
                    reuseNode true
                }
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'd378a78c-6579-4117-b8ba-a6ff9e91cb3a', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        yum install jq -y

                        LATEST_TD_REVISION=$( \
                            aws ecs register-task-definition \
                                --cli-input-json file://aws/task-definition-prod.json \
                                | jq '.taskDefinition.revision' \
                        )
                        
                        echo $LATEST_TD_REVISION
                        aws ecs update-service \
                            --cluster $AWS_ECS_CLUSTER \
                            --service $AWS_ECS_CLUSTER_SERVICE_PROD \
                            --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION

                        aws ecs wait services-stable \
                            --cluster $AWS_ECS_CLUSTER \
                            --services $AWS_ECS_CLUSTER_SERVICE_PROD
                    '''
                }

            }
        }
    }
}
