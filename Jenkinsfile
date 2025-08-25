pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-v $HOME/.npm:/home/node/.npm'
            reuseNode true
        }
    }

    environment {
        npm_config_cache = '/home/node/.npm'
        HOME = '/home/node'
    }

    stages {
        stage('Install') {
            steps {
                sh '''
                    echo "Installing dependencies"
                    npm ci
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running tests"
                    npm test -- --watchAll=false
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Building React app"
                    node --version
                    npm --version
                    npm run build
                '''
                sh '''
                    [ -d build ] || { echo "Build failed: No build folder found"; exit 1; }
                    ls -la build
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'build/**', allowEmptyArchive: false
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
    }
}