pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'f3ebd112-63a7-4d35-9305-fdb149ac5caf'
        NETLIFY_AUTH_TOKEN =  credentials('netlify-token')
    }

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
                        echo "This is the Test stage"
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright LocalTest', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging'){
                    agent{
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        echo "This is the Deploy Staging Stage"
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "Deploying to Staging environment Site Id :- $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build
                        '''
                    
                    }
                }

        stage('Approval'){
                    steps {
                        echo 'Waiting For Approval'
                        timeout(time: 1, unit: 'MINUTES') {
                             input 'Do you want to go For Deployment ?'
                        }
                        
                        
                    }
        }
        
        stage('Deploy Production'){
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "This is the Deploy Production Stage"
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to Production environment Site Id :- $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''

            }
        }

        stage('Production E2E'){
                agent{
                    docker {
                        // image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                        
                    }
                }

                environment{
                    CI_ENVIRONMENT_URL = 'https://flourishing-sunshine-08d68d.netlify.app'
                }

                steps {
                    sh '''
                    echo "This is the End to End Stage for production check "
                    npx playwright test --reporter=line
                    '''
                }
                post{
                    always{
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright ProductionTest ', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
            } 
    } 
}
