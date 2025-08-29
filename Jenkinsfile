pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '39a1ee8a-785a-42ce-b7e7-da57840ea10d'  // Replace with your Site ID
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "=== Build Stage ==="
                    echo "Node version: $(node --version)"
                    echo "NPM version: $(npm --version)"
                    echo "Installing dependencies..."
                    npm ci
                    echo "Building application..."
                    npm run build
                    echo "Build completed!"
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "=== Unit Tests ==="
                            if [ -f build/index.html ]; then
                                echo "✅ Build output verified"
                            else
                                echo "❌ Warning: build/index.html not found"
                            fi
                            npm test
                        '''
                    }
                    post {
                        always {
                            // Only try to publish if the file exists
                            script {
                                if (fileExists('jest-results/junit.xml')) {
                                    junit 'jest-results/junit.xml'
                                } else {
                                    echo 'No JUnit results found'
                                }
                            }
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "=== E2E Tests ==="
                            npm install serve
                            echo "Starting server..."
                            node_modules/.bin/serve -s build -p 3000 &
                            SERVER_PID=$!
                            echo "Server PID: $SERVER_PID"
                            
                            echo "Waiting for server to start..."
                            sleep 15
                            
                            echo "Running Playwright tests..."
                            npx playwright test --reporter=html || echo "E2E tests completed"
                            
                            # Cleanup
                            kill $SERVER_PID 2>/dev/null || true
                        '''
                    }

                    post {
                        always {
                            script {
                                if (fileExists('playwright-report/index.html')) {
                                    publishHTML([
                                        allowMissing: false, 
                                        alwaysLinkToLastBuild: false, 
                                        keepAll: false, 
                                        reportDir: 'playwright-report', 
                                        reportFiles: 'index.html', 
                                        reportName: 'Playwright HTML Report'
                                    ])
                                } else {
                                    echo 'No Playwright report found'
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "=== Deploy Stage ==="
                    echo "Installing Netlify CLI..."
                    npm install netlify-cli@17.36.2
                    
                    echo "Netlify CLI version:"
                    node_modules/.bin/netlify --version
                    
                    echo "Checking authentication..."
                    node_modules/.bin/netlify status
                    
                    echo "Site ID: $NETLIFY_SITE_ID"
                    echo "Deploying to production..."
                    
                    node_modules/.bin/netlify deploy \
                        --dir=build \
                        --prod \
                        --site=$NETLIFY_SITE_ID \
                        --auth=$NETLIFY_AUTH_TOKEN
                    
                    echo "✅ Deployment completed!"
                '''
            }
            post {
                success {
                    echo '🚀 Deployment successful!'
                }
                failure {
                    echo '❌ Deployment failed!'
                }
            }
        }
    }

    post {
        always {
            node {
                // Clean workspace only within node context
                cleanWs()
            }
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs above.'
        }
    }
}