pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-u 1000:1000 -v ${WORKSPACE}/.npm:/home/node/.npm'
            reuseNode true
        }
    }

    environment {
        npm_config_cache = "${WORKSPACE}/.npm"
        HOME = "${WORKSPACE}"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                sh 'rm -rf node_modules package-lock.json'
            }
        }

        stage('Install') {
            steps {
                sh '''
                    echo "Updating npm"
                    npm install -g npm@latest
                    echo "Installing dependencies"
                    npm ci --loglevel=verbose || { echo "npm ci failed"; exit 1; }
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
        always {
            archiveArtifacts artifacts: 'npm-debug.log', allowEmptyArchive: true
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
    }
}