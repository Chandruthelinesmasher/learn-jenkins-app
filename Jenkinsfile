pipeline {
    agent any

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                echo '🛠️ Starting Build Stage...'
                sh 'ls -la'
                sh 'node --version'
                sh 'npm --version'
                sh 'npm ci'
                sh 'npm run build'
                echo '✅ Build completed successfully'
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                        }
                    }
                    steps {
                        echo '🧪 Running Unit Tests...'
                        sh 'npm test'
                    }
                    post {
                        always {
                            echo '📄 Publishing Unit Test Results...'
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        }
                    }
                    steps {
                        echo '🎭 Running End-to-End Tests...'
                        sh 'npm install serve'
                        sh 'sleep 10 & npx serve -s build & npx playwright test --reporter=html'
                    }
                    post {
                        always {
                            echo '📊 Publishing Playwright HTML Report...'
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

        stage('Deploy') {
            agent {
                docker {
                    // Use full Debian-based Node image instead of Alpine
                    image 'node:18'
                }
            }
            steps {
                echo '🚀 Starting Deployment Stage...'
                sh '''
                    apt-get update && apt-get install -y python3 make g++ curl
                    npm install -g netlify-cli@20.1.1
                    netlify deploy --dir=build --prod --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                '''
            }
        }
    }

    post {
        always {
            echo '🏁 Pipeline execution completed!'
        }
    }
}
