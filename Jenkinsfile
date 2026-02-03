pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '77e66bee-2240-44d8-b7e8-1ec23420c087'
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

        stage('Test') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }    
                    }

                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''

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
                            npm install -g serve
                            serve -s build & 
                            sleep 10
                            npx playwright test
                        '''

                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }    
            }
            steps {
                sh '''
                    npm install -g netlify-cli
                    netlify --version
                    echo "Deploying... Site: ID: $NETLIFY_SITE_ID"
                '''
            }
        }
    }
}