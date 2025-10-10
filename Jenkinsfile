pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root'
                }
            }
            steps {
                sh '''
                    echo "Cleaning node_modules only..."
                    rm -rf node_modules
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
                            args '-u root'
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
                            junit 'junit.xml'  // ✅ Fixed path
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '-u root'
                        }
                    }
                    steps {
                        sh '''
                            npm ci
                            npx serve -s build &
                            echo "Waiting for server to start..."
                            sleep 10  # ✅ Increased wait time
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
                    image 'node:18'  // ✅ Debian-based, not alpine
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