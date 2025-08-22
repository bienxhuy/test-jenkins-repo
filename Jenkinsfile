pipeline {
    agent any

    environment {
        // Use existing Docker Compose network
        DOCKER_NETWORK = 'test_default'
        // Define container names
        DEV_CONTAINER = 'dev-instance'
        TEST_CONTAINER = 'test-instance'
        // Define BASE_URL for test framework (using container name for communication within Docker network)
        BASE_URL = "http://${DEV_CONTAINER}:5174"
        // Define InfluxDB host (using existing container name)
        INFLUXDB_HOST = 'influxdb3'
        INFLUXDB_PORT = '8181'
    }

    stages {
        stage('Deploy Code') {
            steps {
                script {
                    // Pull Node.js Docker image
                    bat 'docker pull node:22.16.0'

                    // Run Node.js container in the existing network
                    bat "docker run -d --name ${DEV_CONTAINER} --network ${DOCKER_NETWORK} -p 5174:5174 -v %WORKSPACE%:/app -w /app node:22.16.0 tail -f /dev/null"

                    // Clone the repository inside the container
                    bat "docker exec ${DEV_CONTAINER} git clone -b master https://github.com/bienxhuy/test-instance.git /app/test-instance"

                    // Install dependencies
                    bat "docker exec ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm install\""

                    // Start the server in the background
                    bat "docker exec -d ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm run dev\""

                    // Wait for the server to be ready
                    // bat "docker exec ${DEV_CONTAINER} bash -c \"until curl -s ${BASE_URL}; do echo 'Waiting for server...'; sleep 2; done\""
                }
            }
        }

        stage('Prepare Test Framework') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'log-collector-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                        // Pull Python Docker image (specific version)
                        bat 'docker pull python:3.12.10'

                        // Run Python container in the existing network with BASE_URL environment variable
                        bat "docker run -d --name ${TEST_CONTAINER} --network ${DOCKER_NETWORK} -v %WORKSPACE%:/app -w /app -e BASE_URL=${BASE_URL} python:3.12.10 tail -f /dev/null"

                        // Clone the test framework repository inside the container
                        bat "docker exec ${TEST_CONTAINER} git clone -b dev https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/bienxhuy/devtest.git /app/devtest"

                        // Install Python dependencies
                        bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pip install -r requirements.txt\""
                    }
                }
            }
        }

        stage('Execute Test') {
            steps {
                script {
                    echo 'Executing tests...'
                    // Run pytest in the test framework directory
                    bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pytest\""
                    // Note: Assumes pytest is configured to send results to InfluxDB at influxdb3:8181
                    // If additional arguments are needed, e.g., --influxdb-host=${INFLUXDB_HOST} --influxdb-port=${INFLUXDB_PORT}, add them here
                }
            }
        }

        stage('Collect Test Results') {
            steps {
                echo 'Collecting test results...'
                // Add steps to collect and process test results from the test instance
                // Example: Copy results from test container to workspace
                // bat "docker cp ${TEST_CONTAINER}:/app/devtest/results %WORKSPACE%/results"
            }
        }
    }

    post {
        always {
            echo 'Stopping services and cleaning up...'
            script {
                // Stop and remove only the dev and test containers
                bat "docker stop ${DEV_CONTAINER} || echo 'Error stopping ${DEV_CONTAINER}'"
                bat "docker rm ${DEV_CONTAINER} || echo 'Error removing ${DEV_CONTAINER}'"
                bat "docker stop ${TEST_CONTAINER} || echo 'Error stopping ${TEST_CONTAINER}'"
                bat "docker rm ${TEST_CONTAINER} || echo 'Error removing ${TEST_CONTAINER}'"
                // Do not remove the existing Docker network or InfluxDB/Grafana containers
                // Clean Jenkins workspace
                cleanWs()
            }
        }
    }
}