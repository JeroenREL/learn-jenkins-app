pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c48b3e51-a0ac-42e3-96b9-a401d1dc4a4c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        //variable names have to be correct, so Netlify recognizes it
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
                //"-t my-playwright" assigns the tag "my-playwright" ot the build
                //"." says that it can be found in this current location / root of this project (general Linux command)
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
                            //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            image 'my-playwright'
                            // img changed due to use of custom docker image
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #npm install serve
                            #=> now in custom docker img
                            serve -s build & #the & makes the server run in the background, so it doesn't block following commands
                            # original path "node_modules/.bin/serve -s ..." now simpler due to custom docker img build
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
        stage('Staging deploy & test') {
            agent {
                docker {
                    image 'my-playwright'
                    //original img "mcr.microsoft.com/playwright:v1.39.0-jammy" is now defined in docker img "my-playwright" 
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
                //initialize var so that playwright sees it (and uses this var iso localhost:3000 - see config) 
            }
            steps {
                sh '''
                    #npm install netlify-cli node-jq
                    #=> part of docker build now
                    #node_modules/.bin/netlify --version
                    netlify --version
                    #full path no longer needed: see below
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    #node_modules/.bin/netlify status
                    netlify status
                    #node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    netlify deploy --dir=build --json > deploy-output.json
                    #full path no longer needed, since it is in docker build, which refers to the root of this project
                    # without 'prod' flag, folder 'build' will be deployed to local env ~ staging env
                    CI_ENVIRONMENT_URL=$(
                    #node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                    node-jq -r '.deploy_url' deploy-output.json
                    #simpler path due to docker build
                    )
                    # => part within parenthesis (now stored to the  var**) was moved moved to part of variable script
                    # **(only accessible in the 'sh' block)
                    # => that was when stage deploy and stage test were still two separate stages!
                    # => they were 'merged' later on
                    # parse the value from the key 'deploy_url' from the json file by using .../.bin/node-jq
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
        // stage('Approval') {
        //     steps {
        //         timeout(time: 15, unit: 'MINUTES') {
        //             input 'Ready to deploy?'
        //         }
        //     }
        // }
        // stage('Deploy prod') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             npm install netlify-cli
        //             node_modules/.bin/netlify --version
        //             echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //             node_modules/.bin/netlify status
        //             node_modules/.bin/netlify deploy --dir=build --prod
        //             # deploy the folder 'build' to production
        //         '''
        //     }
        // }
        stage('Prod deploy & test') {
            agent {
                docker {
                    //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    //now part of docker build img "my-playwright"
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://bright-cuchufli-33b1f5.netlify.app'
            }
            steps {
                sh '''
                    #npm install netlify-cli
                    #=>docker
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    # deploy the folder 'build' to production
                    # simpler paths due to docker img (see also staging build stage)
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
