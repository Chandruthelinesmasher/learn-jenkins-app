pipeline {
    agent any

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
                sh '''
                    echo "🛠️ Starting Build Stage..."
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    echo "✅ Build completed successfully"
                    ls -la
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
                        sh '''
                            echo "🧪 Running Unit Tests..."
                            npm test
                        '''
                    }
                    post {
                        always {
                            echo "📄 Publishing Unit Test Results..."
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
                        sh '''
                            echo "🎭 Running End-to-End Tests..."
                            npm install serve
                            npx serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            echo "📊 Publishing Playwright HTML Report..."
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
                    image 'node:18-alpine'
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "🚀 Starting Deployment Stage..."
                    npm install netlify-cli@20.1.1
                    npx netlify --version
                    echo "✅ Netlify CLI Installed Successfully"
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above for errors."
        }
    }
}
