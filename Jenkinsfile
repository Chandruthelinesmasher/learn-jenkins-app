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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
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
                    echo "Test stage"
                    if [ -f build/index.html ]; then
                        echo "✅ index.html exists in build directory"
                    else
                        echo "❌ index.html not found in build directory"
                        exit 1
                    fi

                    echo "Running unit tests..."
                    npm test
                '''
            }
        }
    }
}