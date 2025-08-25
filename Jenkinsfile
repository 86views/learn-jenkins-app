pipeline {
    agent any

    stages {
        stage('Build and Test') {
              agent {
                docker {
                    image 'node:18-alpine' // Use Node.js 18 Alpine for lightweight container
                    reuseNode true
                    args '-v /home/jenkins/.npm:/home/jenkins/.npm' // Cache npm dependencies
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
    }
}
