pipeline {
    agent any

    stages {
        // comment type1: This is a comment
        /*
        comment type2
        Line 1
        line 2
        
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
        */

        stage('Test') {
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
        }


    }

    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwirght HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
