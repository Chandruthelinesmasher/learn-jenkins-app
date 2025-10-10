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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
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
                            npm test -- --ci --reporters=default --reporters=jest-junit
                        '''
                    }
                    post {
                        always {
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
                            npm install serve
                            nohup npx serve -s build > serve.log 2>&1 &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
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
                    image 'node:18'
                    args '-u root:root'
                    reuseNode true
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('netlify-token')
                NETLIFY_SITE_ID = '74da41cd-1adf-407d-935e-94ee5855477c'
            }
            steps {
                sh '''
                    echo "🚀 Installing Netlify CLI..."
                    npm install -g netlify-cli@20.1.1

                    echo "🔑 Deploying to Netlify..."
                    netlify deploy \
                      --dir=build \
                      --prod \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID

                    echo "✅ Deployment successful!"
                '''
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline completed successfully — deployed to Netlify!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs above.'
        }
    }
}
