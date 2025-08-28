pipeline {
    agent any

    environment {
        // Define environment variables for consistency
        NODEJS_HOME = tool name: 'Node18', type: 'NodeJS'
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-v ${HOME}/.npm:/root/.npm' // Use HOME to avoid hardcoding
                }
            }
            steps {
                sh '''
                    echo "Workspace contents:"
                    ls -la
                    node --version
                    npm --version
                    npm ci --prefer-offline --cache=/root/.npm
                    npm run build
                    echo "Build output:"
                    ls -la
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-v ${HOME}/.npm:/root/.npm'
                }
            }
            steps {
                sh '''
                    npm run test || { echo "Tests failed"; exit 1; }
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            // Clean up workspace to free disk space
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}