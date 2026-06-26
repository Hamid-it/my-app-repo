pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('List Files') {
      steps {
        sh 'pwd'
        sh 'ls -la'
      }
    }

    stage('Validate') {
      steps {
        sh 'test -f index.html'
        sh 'test -f Dockerfile'
        echo 'Basic validation passed'
      }
    }
  }
}