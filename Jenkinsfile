pipeline {
    agent any
    stages {
        stage('Build') {
            agent {
                docker {
                     // image 'node:18-alpine'  // make sure it's the correct image namedd
                    image 'node:18'
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
                '''
            }
        }
    }
}