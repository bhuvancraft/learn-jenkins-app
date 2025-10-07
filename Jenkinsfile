pipeline {
    agent any

    environment {
        CI = 'true'
        NETLIFY_SITE_ID = '01683cc0-5dc7-4feb-bc3e-e3ff906f83cf'
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
                    echo "📦 Installing dependencies and building React app..."
                    npm ci
                    npm run build
                    ls -la build
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '-v $WORKSPACE/test-results:/test-results'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "🧪 Running Jest Unit Tests..."
                            mkdir -p /test-results
                            npm ci
                            JEST_JUNIT_OUTPUT_DIR=/test-results npm test -- --watchAll=false
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            args '-v $WORKSPACE:/app -w /app'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "🎭 Running Playwright E2E Tests..."
                            npm ci
                            npm install serve
                            nohup npx serve -s build & sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML(target: [
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report'
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy to Netlify') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "🚀 Deploying build folder to Netlify..."
                    npm install -g netlify-cli
                    netlify deploy --dir=build --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN --prod
                    echo "✅ Deployment completed successfully!"
                '''
            }
        }
    }

    post {
        success {
            echo "
