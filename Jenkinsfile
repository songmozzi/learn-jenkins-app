pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'b609d8a9-29e5-4f8b-9c8c-8f3ee686aa15'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }


    stages {
        // comment type1: This is a comment
        
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'Small change'
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
                            /* bad case
                            args '-u root:root'
                            */

                        }
                    }
                    
                    steps {
                        // not use root, can install local
                        sh ''' 
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwirght Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
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
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to produection. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    /* bad case
                    args '-u root:root'
                    */

                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://astonishing-twilight-731ccc.netlify.app'
            }
            
            steps {
                // not use root, can install local
                sh ''' 
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwirght E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }                    
        }  
    }
}
