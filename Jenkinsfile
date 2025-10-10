pipeline {
    agent any

    environment {
        // Securely load Netlify credentials from Jenkins
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')   // Add this in Jenkins Credentials
        NETLIFY_SITE_ID    = credentials('netlify-site-id')      // Add this in Jenkins Credentials
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                echo '🛠️ Starting Build Stage...'
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    echo "✅ Build completed successfully"
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '-u root:root'
                            reuseNode true
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
                            args '-u root:root'
                            reuseNode true
                        }
                    }
                    steps {
                        echo '🎭 Running End-to-End Tests...'
                        sh '''
                            npm install serve
                            npx serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            echo '📊 Publishing Playwright HTML Report...'
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
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
            agent {
                docker {
                    // Use Debian-based Node image for proper build tools
                    image 'node:18'
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                echo '🚀 Starting Deployment Stage...'
                sh '''
                    apt-get update && apt-get install -y python3 make g++ curl
                    npm install -g netlify-cli@20.1.1

                    echo "🔑 Authenticating and deploying to Netlify..."
                    netlify deploy \
                      --dir=build \
                      --prod \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID

                    echo "✅ Deployment to Netlify successful!"
                '''
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check the logs above for details.'
        }
    }
}
