pipeline {
    agent {
        docker {
            image 'node:18'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}
