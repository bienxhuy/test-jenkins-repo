pipeline {
    agent any
    stages {
        stage('No-op') {
            steps {
                bat 'dir'
            }
        }
    }
    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir() /* clean up our workspace */
            mail to: 'bienxhuy@gmail.com',
                 subject: 'Jenkins Build Status',
                 body: "The build has finished with status: ${currentBuild.currentResult}"
        }
        success {
            echo 'I succeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }
}