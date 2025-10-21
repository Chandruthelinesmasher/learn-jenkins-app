pipeline {
    agent any

    environment {
        NODE_ENV = 'development'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Chandruthelinesmasher/learn-jenkins-app.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm run build'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}