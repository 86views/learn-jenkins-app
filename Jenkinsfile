pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '39a1ee8a-785a-42ce-b7e7-da57840ea10d'  // Replace with your Netlify site ID
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CI = 'true'  // Helps with various npm behaviors in CI
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
                    echo "Node.js version: $(node --version)"
                    echo "NPM version: $(npm --version)"
                    echo "Installing dependencies..."
                    npm ci
                    echo "Building application..."
                    npm run build
                    echo "Build completed. Contents:"
                    ls -la build/ || ls -la dist/ || echo "Build directory not found"
                '''
            }
            post {
                success {
                    // Archive build artifacts
                    archiveArtifacts artifacts: 'build/**/*', fingerprint: true, allowEmptyArchive: true
                }
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
                            echo "Running unit tests..."
                            # Verify build output exists
                            if [ -f build/index.html ]; then
                                echo "Build output verified: build/index.html exists"
                            else
                                echo "Warning: build/index.html not found"
                            fi
                            
                            # Run tests with proper reporters
                            npm test -- --ci --watchAll=false --testResultsProcessor=jest-junit
                        '''
                    }
                    post {
                        always {
                            // Publish test results
                            junit 'junit.xml'
                            // Publish coverage if available
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
                                reportDir: 'coverage/lcov-report',
                                reportFiles: 'index.html',
                                reportName: 'Coverage Report'
                            ])
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
                            echo "Setting up E2E tests..."
                            npm install serve
                            
                            echo "Starting application server..."
                            node_modules/.bin/serve -s build -p 3000 &
                            SERVER_PID=$!
                            
                            echo "Waiting for server to be ready..."
                            # Better wait strategy with timeout
                            for i in {1..30}; do
                                if curl -f http://localhost:3000 >/dev/null 2>&1; then
                                    echo "Server is ready!"
                                    break
                                fi
                                echo "Waiting for server... attempt $i/30"
                                sleep 2
                            done
                            
                            echo "Running Playwright tests..."
                            npx playwright test --reporter=html,junit
                            
                            # Clean up
                            kill $SERVER_PID || true
                        '''
                    }

                    post {
                        always {
                            // Publish test results
                            junit 'test-results/junit.xml'
                            // Publish HTML report
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright E2E Report'
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                // Only deploy on main/master branch
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Installing Netlify CLI..."
                    npm install netlify-cli@17.36.2  # Use a more stable version
                    
                    echo "Netlify CLI version:"
                    node_modules/.bin/netlify --version
                    
                    echo "Checking Netlify authentication..."
                    node_modules/.bin/netlify status
                    
                    echo "Deploying to production..."
                    echo "Site ID: $NETLIFY_SITE_ID"
                    
                    # Deploy the build directory
                    node_modules/.bin/netlify deploy \
                        --dir=build \
                        --prod \
                        --site=$NETLIFY_SITE_ID \
                        --auth=$NETLIFY_AUTH_TOKEN
                    
                    echo "Deployment completed successfully!"
                '''
            }
            post {
                success {
                    echo 'Deployment successful! 🚀'
                }
                failure {
                    echo 'Deployment failed! Check logs for details.'
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}