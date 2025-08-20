pipeline {
    agent {
        docker { image 'node:22.18.0-alpine3.22' }
    }
    stages {
        stage('Test') {
            steps {
                bat 'node --eval "console.log(process.arch,process.platform)"'
            }
        }
    }
}