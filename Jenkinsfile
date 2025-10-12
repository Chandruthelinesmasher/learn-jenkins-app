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
                    echo '🔧 Build stage started'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la build/
                '''
                // Stash the build output and node_modules for reuse
                stash includes: 'build/**/*', name: 'build-artifacts'
                stash includes: 'node_modules/**/*', name: 'node-modules'
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '-u root:root'
                            reuseNode true
                        }
                    }
                    steps {
                        // Reuse node_modules from build stage
                        unstash 'node-modules'
                        sh '''
                            echo '🧪 Running unit tests...'
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
                            reuseNode true
                        }
                    }
                    steps {
                        unstash 'node-modules'
                        unstash 'build-artifacts'
                        sh '''
                            echo '🧪 Running E2E tests...'
                            nohup npx serve -s build -l 3000 > serve.log 2>&1 &
                            sleep 10
                            # Verify server is running
                            curl -f http://localhost:3000 || (cat serve.log && exit 1)
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
                    reuseNode true
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('netlify-token')
                // Replace with your actual Netlify site ID (UUID format)
                NETLIFY_SITE_ID = credentials('netlify-site-id')
            }
            steps {
                unstash 'build-artifacts'
                sh '''
                    echo "🚀 Installing system dependencies for sharp..."
                    apk add --no-cache vips-dev fftw-dev build-base python3

                    echo "🚀 Installing Netlify CLI..."
                    npm install -g netlify-cli@20.1.1

                    echo "🚀 Deploying to Staging..."
                    netlify deploy \
                      --dir=build \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID \
                      --json > deploy-output.json

                    # Extract and save the deploy URL
                    cat deploy-output.json
                    DEPLOY_URL=$(cat deploy-output.json | grep -o '"deploy_url":"[^"]*' | cut -d'"' -f4)
                    echo "Deploy URL: $DEPLOY_URL"
                    echo "$DEPLOY_URL" > staging-url.txt

                    echo "✅ Staging deployment complete!"
                '''
                stash includes: 'staging-url.txt', name: 'staging-url'
            }
        }

        stage('Staging Smoke Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                unstash 'staging-url'
                unstash 'node-modules'
                sh '''
                    echo '🧪 Running smoke tests on staging...'
                    STAGING_URL=$(cat staging-url.txt)
                    echo "Testing URL: $STAGING_URL"
                    
                    # Basic health check
                    curl -f $STAGING_URL || exit 1
                    
                    # Optional: Run Playwright tests against staging URL
                    # BASE_URL=$STAGING_URL npx playwright test --reporter=html
                    
                    echo "✅ Staging smoke tests passed!"
                '''
            }
        }

        stage('Approval for Prod Deploy') {
            steps {
                script {
                    timeout(time: 24, unit: 'HOURS') {
                        input message: '🚦 Ready to deploy to production. Approve?', ok: 'Deploy Now'
                    }
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                    reuseNode true
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('netlify-token')
                NETLIFY_SITE_ID = credentials('netlify-site-id')
            }
            steps {
                unstash 'build-artifacts'
                sh '''
                    echo "🚀 Installing system dependencies for sharp..."
                    apk add --no-cache vips-dev fftw-dev build-base python3

                    echo "🚀 Installing Netlify CLI..."
                    npm install -g netlify-cli@20.1.1

                    echo "🔑 Deploying to Production..."
                    netlify deploy \
                      --dir=build \
                      --prod \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID \
                      --json > prod-deploy-output.json

                    # Extract and save the production URL
                    cat prod-deploy-output.json
                    PROD_URL=$(cat prod-deploy-output.json | grep -o '"url":"[^"]*' | cut -d'"' -f4)
                    echo "Production URL: $PROD_URL"
                    echo "$PROD_URL" > prod-url.txt

                    echo "✅ Production deployment successful!"
                '''
                stash includes: 'prod-url.txt', name: 'prod-url'
            }
        }

        stage('Prod E2E') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                unstash 'prod-url'
                unstash 'node-modules'
                sh '''
                    echo '🧪 Running Production E2E tests...'
                    PROD_URL=$(cat prod-url.txt)
                    echo "Testing Production URL: $PROD_URL"
                    
                    # Health check
                    curl -f $PROD_URL || exit 1
                    
                    # Run Playwright tests against production
                    BASE_URL=$PROD_URL npx playwright test --reporter=html
                    
                    echo "✅ Production E2E tests passed!"
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
                failure {
                    echo '⚠️ Production E2E tests failed, but deployment is complete'
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
        cleanup {
            cleanWs()
        }
    }
}