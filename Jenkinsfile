pipeline {
    agent any

    environment {
        DEV_REPO   = 'https://github.com/bienxhuy/test-instance.git'
        DEV_BRANCH = 'master'
        TEST_REPO  = 'https://github.com/bienxhuy/devtest.git'
        TEST_BRANCH = 'dev'
    }

    stages {
        stage('Deploy Code') {
            steps {
                echo 'Cloning product repo...'
                dir('dev-app') {
                    git branch: "${DEV_BRANCH}", url: "${DEV_REPO}"
                }
            }
        }

        stage('Prepare Test Framework') {
            steps {
                echo 'Cloning test framework repo...'
                dir('test-app') {
                    git branch: "${TEST_BRANCH}", url: "${TEST_REPO}"
                }
            }
        }

        stage('Execute Test') {
            steps {
                echo 'Running containers with docker-compose...'
                script {
                    // Bring up dev + test containers
                    bat 'docker compose up --abort-on-container-exit --exit-code-from tests'
                }
            }
        }

        stage('Collect Test Results') {
            steps {
                echo 'Collecting test results...'
            }
        }
    }

    post {
        always {
            echo 'Stopping services and cleaning up...'
            bat 'docker compose down -v'
            cleanWs()
        }
    }
}
