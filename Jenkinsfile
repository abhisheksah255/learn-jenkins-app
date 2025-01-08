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

        stage(Tests) {
            parallel{
                stage('Unit Test'){
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
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E'){
                    agent{
                        docker {
                            // image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            
                        }
                    }
                    steps {
                        sh '''
                        echo "This is the E2E Stage"
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=line
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
       stage('Deploy'){
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "This is the Deploy Stage"
                npm install netlify-cli -g
                netlify --version
                ls -la
                '''
            
            }
        } 
    } 
}
