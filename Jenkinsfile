pipeline {
    agent any

    environment { 
        AWS_DEFAULT_REGION="us-east-1"
    }

    stages {
        stage("Deploy prod"){
            agent {
                docker {
                    image "amazon/aws-cli"
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'd378a78c-6579-4117-b8ba-a6ff9e91cb3a', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        yum install jq -y

                        LATEST_TD_REVISION=$(aws ecs register-task-definition \
                            --cli-input-json file://aws/task-definition-prod.json \
                            | jq '.taskDefinition.revision')
                        
                        echo $LATEST_TD_REVISION
                        aws ecs update-service \
                            --cluster LearnJenkinsApp-Cluster-prod \
                            --service LearnJenkinApp-Service-Prod \
                            --task-definition LearnJenkinsApp-TaskDeifinition-Prod:$LATEST_TD_REVISION

                        aws ecs wait services-stable \
                            --cluster LearnJenkinsApp-Cluster-prod \
                            --services LearnJenkinApp-Service-Prod
                    '''
                }

            }
        }

        // stage('Build') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }
    }
}
