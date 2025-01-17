pipeline {
    agent any

    environment { 
        NETLIFY_SITE_ID = 'c57261eb-c8bf-40f5-9297-aacc858aee8b'
    }

    stages {

        stage('Docker'){
            steps {
                sh "docker build -t my-playwright ."
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
        
        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwrigh'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                '''
            }
        }
        stage('Approval stage -> prod') {

            steps {
                timeout(activity: true, time: 15) {
                    input message: 'Approved?', ok: 'Approved'
                }
                
            }
        }
        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwrigh'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                '''
            }
        }
    }
}
