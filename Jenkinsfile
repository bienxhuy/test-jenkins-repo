pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                bat 'python --version'
            }
        }
    }
    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'Pipeline succeeded'
        }
        failure {
            echo 'Pipeline failed'
        }
        unstable {
            echo 'Pipeline is unstable'
        }
        changed {
            echo 'This will run only when state of run changed'
            echo 'For example: if Pipeline was previously failing but is now successful'
        }
    }
}