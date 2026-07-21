pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = "0efcdaec-a4f4-47e2-8d43-59ae31af3a17"
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
                    echo "Build Stage"
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Run Tests') {
            parallel {

                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "Running Unit Tests"
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('Local E2E Test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "Running Local E2E Tests"

                            npm install
                            npm install serve

                            npx serve -s build -l 3000 &
                            sleep 10

                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                icon: '',
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright Local Report',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo "Deploying to Staging"

                    npx netlify-cli --version
                    npx netlify-cli status
                    npx netlify-cli deploy --dir=build
                '''
            }
        }

        stage('Approval') {
            steps {
                input message: 'Do you wish to deploy to production', ok: 'Yes, I am sure!'
            }
            
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo "Deploying to Netlify"

                    npx netlify-cli --version
                    npx netlify-cli status
                    npx netlify-cli deploy --dir=build --prod
                '''
            }
        }

        stage('PROD E2E Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "https://playful-medovik-ab5931.netlify.app"
            }

            steps {
                sh '''
                    echo "Running Production E2E Tests"

                    npm install

                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        icon: '',
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Production Report',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'jest-results/junit.xml'

            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: false,
                icon: '',
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report',
                reportTitles: '',
                useWrapperFileDirectly: true
            ])
        }
    }
}