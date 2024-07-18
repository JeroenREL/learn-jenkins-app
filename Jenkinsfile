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
                sh '''
                #comment: if you run test stage before build, you need to run 'npm ci' at the start of the test stage!
                echo 'Test stage'
                test -f build/index.html
                npm test
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
