pipeline {
    agent any

    stages {
        stage('Build') {
              agent {
                docker {
                    image 'node:18-alpine' 
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
