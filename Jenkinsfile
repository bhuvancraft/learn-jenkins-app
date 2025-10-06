pipeline {
    agent any

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
                    echo 'Building the application...'
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
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo 'Running Unit Tests...'
                            npm ci
                            npm test
                        '''
                    }
                    post {
                        always {
                            // Updated path to match package.json jest-junit config
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo 'Starting E2E Tests...'
                            npm ci
                            npm install serve
                            node_modules/.bin/serve -s build -l 3000 &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false,
                                         alwaysLinkToLastBuild: false,
                                         keepAll: false,
                                         reportDir: 'playwright-report',
                                         reportFiles: 'index.html',
                                         reportName: 'Playwright HTML Report',
                                         useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
