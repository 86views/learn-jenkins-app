pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-v $HOME/.npm:/home/jenkins/.npm' // Cache node_modules
            reuseNode true
        }
    }

    environment {
        npm_config_cache = '/home/jenkins/.npm'
        HOME = '/home/jenkins'
    }

    stages {
        stage('Build') {
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
                    npm --version
                    npm run build
                '''

                sh 'ls -la build || echo "No build folder found"'
            }
        }
    }
}
