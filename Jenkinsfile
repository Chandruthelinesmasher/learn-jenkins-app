pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    echo '🔧 Build stage started'
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
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                            echo '🧪 Running unit tests...'
                            npm ci
                            npm test -- --ci --reporters=default --reporters=jest-junit
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                            echo '🧪 Running E2E tests...'
                            npm ci
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

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('netlify-token')
                NETLIFY_SITE_ID = 'nfp_X7f3qYEMaNRpmRRdzi8P9bAhE1pwSX4R662b'
            }
            steps {
                sh '''
                    echo "🚀 Installing system dependencies for sharp..."
                    apk add --no-cache vips-dev fftw-dev build-base python3

                    echo "🚀 Installing Netlify CLI..."
                    npm install -g netlify-cli@20.1.1

                    echo "🚀 Deploying to Staging..."
                    netlify deploy \
                      --dir=build \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID

                    echo "✅ Staging deployment complete!"
                '''
            }
        }

        stage('Approval for Prod Deploy') {
            steps {
                input message: '🚦 Ready to deploy to production. Approve?', ok: 'Deploy Now'
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('netlify-token')
                NETLIFY_SITE_ID = 'nfp_X7f3qYEMaNRpmRRdzi8P9bAhE1pwSX4R662b'
            }
            steps {
                sh '''
                    echo "🚀 Installing system dependencies for sharp..."
                    apk add --no-cache vips-dev fftw-dev build-base python3

                    echo "🚀 Installing Netlify CLI..."
                    npm install -g netlify-cli@20.1.1

                    echo "🔑 Deploying to Netlify..."
                    netlify deploy \
                      --dir=build \
                      --prod \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID

                    echo "✅ Production deployment successful!"
                '''
            }
            post {
                unsuccessful {
                    script {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    echo '🧪 Running Prod E2E tests...'
                    npm ci
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
                        reportName: 'Prod Playwright HTML Report'
                    ])
                }
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