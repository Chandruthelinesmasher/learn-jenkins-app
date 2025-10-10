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
                    args '-u root'  // Optional: if you face permission issues with reports
                }
            }
            steps {
                sh '''
                    npm ci --prefer-offline

                    # Serve the built app
                    npx serve -s build &
                    echo "Waiting for server to start..."
                    sleep 5

                    # Run Playwright tests
                    npx playwright test --reporter=html,junit
                '''
            }
        }
    }

    post {
        always {
            // Publish JUnit results (from Test or E2E)
            script {
                if (fileExists('test-results/junit.xml')) {
                    junit 'test-results/junit.xml'
                } else {
                    echo 'No JUnit report found at test-results/junit.xml'
                }
            }

            // Publish Playwright HTML report
            publishHTML([
                allowMissing: true,  // Don't fail if report missing
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Report'
            ])
        }
    }
}