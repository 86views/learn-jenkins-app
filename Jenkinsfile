pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
            args '-w /home/jenkins/workspace/$JOB_NAME'
        }
    }

    environment {
         NETLIFY_SITE_ID = '39a1ee8a-785a-42ce-b7e7-da57840ea10d'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')

        // NETLIFY_SITE_ID    = 'YOUR_SITE_ID'                      // replace with your Netlify site ID
        // NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')   // Jenkins credential ID
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la build
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    steps {
                        sh '''
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'  // requires jest-junit config
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '-w /home/jenkins/workspace/$JOB_NAME'
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production..."
                    node_modules/.bin/netlify deploy \
                        --dir=build \
                        --prod \
                        --auth=$NETLIFY_AUTH_TOKEN \
                        --site=$NETLIFY_SITE_ID
                '''
            }
        }
    }
}
