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
        stage('Test stage') {
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
        stage('E2E') {
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
                    node_modules/.bin/serve -s build
                    npx playwright test
                '''  
            }
        }
    }
    post{
        always{
            junit 'test-results/junit.xml'
        }
    }
}
