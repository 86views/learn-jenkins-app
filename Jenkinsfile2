pipeline {
    agent any
      
    environment {
        NETLIFY_SITE_ID = '39a1ee8a-785a-42ce-b7e7-da57840ea10d'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine' 
                    reuseNode true
                    args '-v /home/jenkins/.npm:/home/jenkins/.npm'
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
               
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine' 
                    reuseNode true
                    args '-v /home/jenkins/.npm:/home/jenkins/.npm'
                }
            }
            steps {
                sh '''
                    test -f build/index.html
                    npm test
                    ls -la test-results/
                '''
                // Stash the Jest test results
                stash name: 'jest-test-results', includes: 'test-results/junit.xml'
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
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                    ls -la test-results/
                    ls -la playwright-report/
                '''
                // Stash the Playwright test results
                stash name: 'playwright-test-results', includes: 'test-results/playwright-junit.xml'
                stash name: 'playwright-html-report', includes: 'playwright-report/**'
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine' 
                    reuseNode true
                    args '-v /home/jenkins/.npm:/home/jenkins/.npm'
                }
            }
            steps {
                sh '''
                  npm install netlify-cli@20.1.1
                  node_modules/.bin/netlify --version
                  echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                '''
            }
        }
        
        stage('Collect Test Results') {
            agent any
            steps {
                // Unstash all test results to the workspace
                unstash 'jest-test-results'
                unstash 'playwright-test-results'
                unstash 'playwright-html-report'
                // Verify files are available
                sh '''
                    echo "=== Available test result files ==="
                    find . -name "*.xml" -type f
                    ls -la test-results/ || echo "test-results directory not found"
                '''
            }
        }
    }

    post {   
        always {
            // Now the test results should be available in the workspace
            junit 'test-results/junit.xml'
            junit 'test-results/playwright-junit.xml'
            junit 'test-results/**/*.xml'
            
            // HTML report for Playwright
            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Report'
            ])
        }
    }
}