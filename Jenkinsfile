pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    // This approach works better with Docker-in-Docker setups
                    docker.image('node:18').inside('-u root:root') {
                        sh '''
                            echo "=== Environment Check ==="
                            pwd
                            ls -la
                            
                            echo "=== Node.js Version ==="
                            node --version
                            npm --version
                            
                            echo "=== Installing Dependencies ==="
                            npm ci
                            
                            echo "=== Building Application ==="
                            npm run build
                            
                            echo "=== Build Complete ==="
                            ls -la
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo 'Build completed'
        }
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}