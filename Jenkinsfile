pipeline {
    agent any

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
        stage('Test runs') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        //comment in piepline script
                        sh '''
                        #comment in shell: if you run test stage before build, you need to run 'npm ci' at the start of the test stage!
                        echo 'Test stage'
                        test -f build/index.html
                        npm test
                        '''  
                    }
                }
                post{
                    always{
                        junit 'jest-test-results/junit.xml
                    }
                }
                stage('E2E Test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        //comment in piepline script
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build & #the & makes the server run in the background, so it doesn't block following commands
                            sleep 10 #wait 10 secs for the server to start before starting the PW tests
                            npx playwright test --reporter=html
                        '''  
                    }
                }
                post{
                    always{
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        //pipeline script to publish html reports generated in jenkins (pipeline - pipeline syntax)
                    }
                }
            }
        }
    }
}
