pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '77e66bee-2240-44d8-b7e8-1ec23420c087'
        NETLIFY_AUTH_TOKEN =  credentials('netlify-token')   
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        stage('Docker') { 
            steps {
                sh 'docker build -t my-dockerfile .'
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
                    echo "lets see github logs..."
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

        stage('Deploy Staging') {
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
                    echo "Deploying to staging... Site: ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --no-build
                '''
            }
        }

        stage('Deploy Production') {
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
                    netlify status
                    netlify deploy --dir=build --prod --no-build
                '''
            }
        }
    }
}