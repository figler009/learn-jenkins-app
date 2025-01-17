pipeline {
    agent any

    environment { 
        NETLIFY_SITE_ID = 'c57261eb-c8bf-40f5-9297-aacc858aee8b'
    }

    stages {
        stage("AWS"){
            agent {
                docker {
                    image "amazon/aws-cli"
                    args "--entrypoint=''"
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'd378a78c-6579-4117-b8ba-a6ff9e91cb3a', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh """
                        aws --version
                        aws s3 ls
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
        
        stage("Upload Build to S3"){
            agent {
                docker {
                    image "amazon/aws-cli"
                    args "--entrypoint=''"
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'd378a78c-6579-4117-b8ba-a6ff9e91cb3a', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh """
                        aws s3 cp ./build s3://jenkins-site-figler009 --recursive
                    """
                }

            }
        }

        // stage('Tests') {
        //     parallel {
        //         stage('Unit tests') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }

        //             steps {
        //                 sh '''
        //                     #test -f build/index.html
        //                     npm test
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('E2E') {
        //             agent {
        //                 docker {
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }

        //             steps {
        //                 sh '''
        //                     serve -s build &
        //                     sleep 10
        //                     npx playwright test  --reporter=html
        //                 '''
        //             }

        //             post {
        //                 always {
        //                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('Deploy staging') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             netlify --version
        //         '''
        //     }
        // }
        // stage('Approval stage -> prod') {

        //     steps {
        //         timeout(activity: true, time: 15) {
        //             input message: 'Approved?', ok: 'Approved'
        //         }
                
        //     }
        // }
        // stage('Deploy prod') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             netlify --version
        //         '''
        //     }
        // }
    }
}
