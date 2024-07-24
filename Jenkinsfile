pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c48b3e51-a0ac-42e3-96b9-a401d1dc4a4c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        //variable names have to be correct, so Neltify recognizes it
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
                    post{
                        always{
                            junit 'jest-test-results/junit.xml'
                        }
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
                        //comment in pipeline script
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build & #the & makes the server run in the background, so it doesn't block following commands
                            sleep 10 #wait 10 secs for the server to start before starting the PW tests
                            npx playwright test --reporter=html
                        '''  
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local', reportTitles: '', useWrapperFileDirectly: true])
                            //pipeline script to publish html reports generated in jenkins (pipeline - pipeline syntax)
                        }
                    }
                }
            }
        }
        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    # without 'prod' flag, folder 'build' will be deployed to local env ~ staging env
                    # node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json => moved to part of variable script
                    # parse the value from the key 'deploy_url' from the json file by using .../.bin/node-jq
                '''
            }
            scrpit {
                env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
            }
        }
        stage('Staging E2E Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                //comment in pipeline script
                sh '''
                    npx playwright test --reporter=html
                '''  
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                    //pipeline script to publish html reports generated in jenkins (pipeline - pipeline syntax)
                }
            }
        }
        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input 'Ready to deploy?'
                }
            }
        }
        stage('Deploy prod') {
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
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    # deploy the folder 'build' to production
                '''
            }
        }
        stage('Prod E2E Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://bright-cuchufli-33b1f5.netlify.app'
            }
            steps {
                //comment in pipeline script
                sh '''
                    npx playwright test --reporter=html
                '''  
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Production E2E', reportTitles: '', useWrapperFileDirectly: true])
                    //pipeline script to publish html reports generated in jenkins (pipeline - pipeline syntax)
                }
            }
        }
    }
}
