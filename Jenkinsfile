pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = "0efcdaec-a4f4-47e2-8d43-59ae31af3a17"
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        
        // stage('Build') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }
        

        // stage('Run Tests') {
        //     parallel {

        //         stage('Unit Test') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }

        //             steps {
        //                 sh '''
        //                     echo "Test Stage"
        //                     npm test
        //                 '''
        //             }

        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('E2E Test') {
        //             agent {
        //                 docker {
        //                     image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //                     reuseNode true
        //                 }
        //             }

        //             steps {
        //                 sh '''
        //                     echo "E2E Playwright Test Stage"
        //                     npm install serve
        //                     node_modules/.bin/serve -s build &
        //                     sleep 10
        //                     npx playwright test --reporter=html
        //                 '''
        //             }

        //             post {
        //                 always {
        //                     publishHTML([
        //                         allowMissing: false,
        //                         alwaysLinkToLastBuild: false,
        //                         icon: '',
        //                         keepAll: false,
        //                         reportDir: 'playwright-report',
        //                         reportFiles: 'index.html',
        //                         reportName: 'Playwright HTML Report',
        //                         reportTitles: '',
        //                         useWrapperFileDirectly: true
        //                     ])
        //                 }
        //             }
        //         }

        //     }
        // }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npx netlify-cli --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    npx netlify-cli status
                    npx netlify-cli deploy --dir=build --prod
                '''
            }
        }

    }

    post {
        always {
            junit 'jest-results/junit.xml'

            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: false,
                icon: '',
                keepAll: false,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report',
                reportTitles: '',
                useWrapperFileDirectly: true
            ])
        }
    }
}