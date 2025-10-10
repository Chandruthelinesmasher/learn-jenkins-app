pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root'  // ✅ FIX
                }
            }
            steps {
                sh '''
                    echo "Cleaning node_modules..."
                    rm -rf node_modules
                    node --version
                    npm ci
                    npm run build
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
                            args '-u root'  // ✅ FIX
                        }
                    }
                    steps {
                        sh '''
                            npm ci
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'junit.xml'  // ✅ Default path for CRA + jest-junit
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '-u root'  // ✅ Already good, but keep it
                        }
                    }
                    steps {
                        sh '''
                            npm ci
                            npx serve -s build &
                            echo "Waiting for server..."
                            sleep 10  // ✅ Increased wait
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
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
                    image 'node:18'  // ✅ Use Debian (not alpine) for sharp compatibility
                    reuseNode true
                    args '-u root'
                }
            }
            steps {
                sh '''
                    npm ci
                    npm install -g netlify-cli
                    netlify --version
                '''
            }
        }
    }
}