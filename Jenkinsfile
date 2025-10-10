pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root'  // 👈 Fix: run as root to avoid EACCES
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
                    args '-u root'  // 👈 Same fix here
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
                    args '-u root'  // Already correct
                }
            }
            steps {
                sh '''
                    npm ci --prefer-offline

                    npx serve -s build &
                    echo "Waiting for server to start..."
                    sleep 5

                    npx playwright test --reporter=html,junit
                '''
            }
        }
    }

    post {
        always {
            script {
                if (fileExists('test-results/junit.xml')) {
                    junit 'test-results/junit.xml'
                } else {
                    echo 'No JUnit report found at test-results/junit.xml'
                }
            }

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