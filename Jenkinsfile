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
    }
}
