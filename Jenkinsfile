pipeline {
    agent any

    stages {

        stage('Tests') {
            parallel {

                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    environment {
                        // Tell jest-junit where to put the report
                        JEST_JUNIT_OUTPUT_DIR = 'jest-results'
                        JEST_JUNIT_OUTPUT_NAME = 'junit.xml'
                    }

                    steps {
                        sh '''
                            echo "Running Unit Tests..."
                            npm ci
                            npm test
                        '''
                    }

                    post {
                        always {
                            echo "Publishing JUnit test report..."
                            junit 'jest-results/junit.xml'
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
                            echo "Running E2E Tests..."
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            echo "Publishing Playwright HTML report..."
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
    }
}
