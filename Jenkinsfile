pipeline {
    agent { docker { image 'python:3.13.7-alpine3.22' } }
    stages {
        stage('build') {
            steps {
                bat 'python --version'
            }
        }
    }
}
