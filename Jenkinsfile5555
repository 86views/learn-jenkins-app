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
                    # List test results to verify file exists
                    ls -la test-results/ || echo "test-results directory not found"
                '''
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
            # Just run playwright test - the config will handle reporters
            npx playwright test
            # Verify files were created
            ls -la test-results/
            ls -la playwright-report/
        '''
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
    }

    post {   
        always {
            // Correct path for Jest JUnit reports (based on your package.json config)
            junit 'test-results/junit.xml'


            junit 'test-results/playwright-junit.xml'
            
            // Also capture any other potential test result files
            junit '**/test-results/**/*.xml'
            
            // Archive the Playwright HTML reports too
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Report'
            ])
        }
    }
}