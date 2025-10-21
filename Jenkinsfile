pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/Chandruthelinesmasher/learn-jenkins-app.git'
      }
    }
    stage('Build') {
      steps {
        echo 'Installing dependencies...'
        sh 'npm install'
      }
    }
    stage('Test') {
      steps {
        echo 'Running tests...'
        sh 'npm test'
      }
    }
  }
}
