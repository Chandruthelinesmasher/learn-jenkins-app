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
                    node --version
                    npm ci --prefer-offline
                    npm run build
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci --prefer-offline
                    npm test
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
                    npm ci --prefer-offline

                    npx serve -s build &
                    echo "Waiting for server..."
                    sleep 3

                    npx playwright test --reporter=html,junit
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
            publishHTML([
                allowMissing: false,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Report'
            ])
        }
    }
}