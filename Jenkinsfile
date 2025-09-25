pipeline {
    agent any

    environment {
        // Use existing Docker Compose network
        DOCKER_NETWORK = "test_default"
        // Define container names
        DEV_CONTAINER = 'dev-instance'
        DEV_PORT = "4173"
        TEST_CONTAINER = 'test-instance'
        // Base URL of container
        BASE_URL = "http:${DEV_CONTAINER}:${DEV_PORT}"
        // Define InfluxDB host (using existing container name)
        INFLUXDB_HOST = 'http://influxdb3:8181/'
        INFLUXDB_DATABASE = 'testdb'
        INFLUXDB_TOKEN = credentials('influxdb-token')
    }

    stages {
        stage('Prepare Environment') {
            parallel {
                stage('Prepare Dev Instance') {
                    steps {
                        // Pull Node.js Docker image
                        echo 'Pulling Node.js Docker image...'
                        echo '-----------------------------------'
                        bat 'docker pull node:22.16.0'
                        echo 'Node.js image pulled successfully.'
                        echo '-----------------------------------'

                        // Run Node.js container in the existing network
                        echo 'Starting dev instance container...'
                        echo '-----------------------------------'
                        bat "docker run -d --name ${DEV_CONTAINER} --network ${DOCKER_NETWORK} -p ${DEV_PORT}:${DEV_PORT} -v %WORKSPACE%:/app -w /app node:22.16.0 tail -f /dev/null"
                        echo 'Dev instance container started successfully.'
                        echo '-----------------------------------'

                        // Clone the repository inside the container
                        echo 'Cloning repository...'
                        echo '-----------------------------------'
                        bat "docker exec ${DEV_CONTAINER} git clone -b ${BRANCH} https://github.com/bienxhuy/test-instance.git /app/test-instance"
                        echo 'Repository cloned successfully.'
                        echo '-----------------------------------'

                        // Install dependencies
                        echo 'Installing dependencies...'
                        echo '-----------------------------------'
                        bat "docker exec ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm install\""
                        echo 'Dependencies installed successfully.'
                        echo '-----------------------------------'

                        // Build the application
                        echo 'Building application...'
                        echo '-----------------------------------'
                        bat "docker exec ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm run build\""
                        echo 'Application built successfully.'
                        echo '-----------------------------------'

                        // Start the server in the background
                        echo 'Starting dev instance server...'
                        echo '-----------------------------------'
                        bat "docker exec -d ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm run preview -- --host ${DEV_CONTAINER}\""
                        echo 'Dev instance server started successfully.'
                        echo '-----------------------------------'
                    }
                }

                stage('Prepare Test Instance') {
                    steps {
                        // Prepare instance
                        echo 'Running Python-Chrome Docker image...'
                        echo '-----------------------------------'
                        bat "docker run -d --name ${TEST_CONTAINER} -e BASE_URL=${BASE_URL} -e INFLUX_HOST=${INFLUXDB_HOST} -e INFLUX_TOKEN=${INFLUXDB_TOKEN} -e INFLUX_DATABASE=${INFLUXDB_DATABASE} -e BUILD_NUMBER=${env.BUILD_NUMBER} -e BUILD_URL=${env.BUILD_URL} -e BRANCH=${env.BRANCH} -e AUTHOR=${env.AUTHOR} -e HOST=${env.HOST} --network ${DOCKER_NETWORK} -v %WORKSPACE%:/app -w /app python-chrome tail -f /dev/null"
                        echo 'Test instance container started successfully.'
                        echo '-----------------------------------'

                        // Clone the test framework repository using credentials
                        echo 'Cloning test framework repository...'
                        echo '-----------------------------------'
                        withCredentials([usernamePassword(credentialsId: 'devtest', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                            // Clone the test framework repository
                            bat "docker exec ${TEST_CONTAINER} git clone -b dev https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/bienxhuy/devtest.git /app/devtest"
                        }
                        echo 'Test framework repository cloned successfully.'
                        echo '-----------------------------------'

                        // Install Python dependencies
                        echo 'Installing Python dependencies...'
                        echo '-----------------------------------'
                        bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pip install -r requirements.txt\""
                        echo 'Python dependencies installed successfully.'
                        echo '-----------------------------------'
                    }
                }
            }
        }
        
        stage('Execute Test') {
            steps {
                echo 'Starting test execution...'
                echo '-----------------------------------'
                // Execute tests with pytest, passing necessary environment variables
                bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pytest\""
                echo 'Test execution completed.'
                echo '-----------------------------------'
            }
        }

        stage('Post Execution') {
            steps {
                echo 'Starting post execution script...'
                echo '-----------------------------------'
                // Run post execution script
                bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && python ./postexec_main.py\""
                echo 'Post execution script completed.'
                echo '-----------------------------------'
            }
        }
    }

    post {
        // Archive logs if the pipeline fails before reaching the cleanup stage
        failure {
            // Archive logs on failure
            echo 'Pipeline failed. Archiving all logs in in %WORKSPACE%/devtest/logs/logs_data/'
            echo '-----------------------------------'
            archiveArtifacts artifacts: 'devtest/logs/logs_data/**', allowEmptyArchive: true
            echo 'Logs archived successfully.'
            echo '-----------------------------------'

            // Archive screenshots on failure
            echo 'Archiving screenshots in %WORKSPACE%/devtest/reports/screenshots/'
        }
        
        // Always perform cleanup actions after all stages
        cleanup {
            echo 'Pipeline finished.'
            echo '-----------------------------------'
            echo 'Stopping services and cleaning up...'
            echo '-----------------------------------'

            // Stop and remove only the dev and test containers
            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat "docker stop ${DEV_CONTAINER} || echo 'Error stopping ${DEV_CONTAINER}'"
                bat "docker rm ${DEV_CONTAINER} || echo 'Error removing ${DEV_CONTAINER}'"
            }
            echo 'Dev instance container stopped and removed.'
            echo '-----------------------------------'

            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat "docker stop ${TEST_CONTAINER} || echo 'Error stopping ${TEST_CONTAINER}'"
                bat "docker rm ${TEST_CONTAINER} || echo 'Error removing ${TEST_CONTAINER}'"
            }
            echo 'Test instance container stopped and removed.'
            echo '-----------------------------------'

            // Clean Jenkins workspace
            cleanWs()
            echo 'Workspace cleaned.'
            echo '-----------------------------------'
            echo 'Pipeline finished.'
        }
    }
}