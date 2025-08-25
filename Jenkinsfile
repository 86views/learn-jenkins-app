pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine' // Use Node.js 18 Alpine for lightweight container
                    reuseNode true
                    args '-v /home/jenkins/.npm:/home/jenkins/.npm' // Cache npm dependencies
                }
            }
            environment {
                npm_config_cache = '/home/jenkins/.npm' // Set npm cache directory
                HOME = '/home/jenkins' // Avoid permission issues
            }

            steps {
                 sh '''
                echo "Installing dependencies"
                npm ci
                '''
                sh '''
                echo "Running tests"
                npm test -- --watchAll=false
                '''
                sh '''
                echo "Building React app"
                node --version
                npm version
                npm run build
                
                '''
               
                sh 'ls -la build' // Verify build output
            }
        }
    }
}
