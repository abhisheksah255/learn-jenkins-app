pipeline {
    agent any

    stages {
        stage('Build'){
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "This is the Build Stage"
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            
            }
        }

        stage('Test'){
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "This is the Test Build"
                ls -la
                test -f build/index.html
                npm test
                ls -la
                '''
            }
        }

        stage('E2E'){
            agent{
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                    reuseNode true
                    arg '-u root:root'
                }
            }
            steps {
                sh '''
                echo "This is the E2E Stage"
                npm install -g server
                serve -s build
                npx playwright test
                '''
            }
        }
    }
}
