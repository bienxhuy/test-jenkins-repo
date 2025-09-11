pipeline {
    agent any

    environment {
        // Use existing Docker Compose network
        DOCKER_NETWORK = "test_default"
        // Define container names
        DEV_CONTAINER = 'dev-instance'
        DEV_PORT = "4173"
        TEST_CONTAINER = 'test-instance'
        BASE_URL = "http://${DEV_CONTAINER}:${DEV_PORT}"
        // Define InfluxDB host (using existing container name)
        INFLUXDB_HOST = 'http://influxdb3:8181/'
        INFLUXDB_DATABASE = 'testdb'
        INFLUXDB_TOKEN = credentials('influxdb-token')
    }

    stages {
        stage('Deploy Code') {
            steps {
                // Pull Node.js Docker image
                bat 'docker pull node:22.16.0'

                // Run Node.js container in the existing network
                bat "docker run -d --name ${DEV_CONTAINER} --network ${DOCKER_NETWORK} -p ${DEV_PORT}:${DEV_PORT} -v %WORKSPACE%:/app -w /app node:22.16.0 tail -f /dev/null"

                // Clone the repository inside the container
                bat "docker exec ${DEV_CONTAINER} git clone -b master https://github.com/bienxhuy/test-instance.git /app/test-instance"

                // Install dependencies
                bat "docker exec ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm install\""

                // Build the application
                bat "docker exec ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm run build\""

                // Start the server in the background
                bat "docker exec -d ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm run preview -- --host 0.0.0.0\""
            }
        }

        stage('Prepare Test Framework') {
            steps {
                // Prepare instance
                bat "docker run -d --name ${TEST_CONTAINER} -e BASE_URL=${BASE_URL} -e INFLUX_HOST=${INFLUXDB_HOST} -e INFLUX_TOKEN=${INFLUXDB_TOKEN} -e INFLUX_DATABASE=${INFLUXDB_DATABASE} --network ${DOCKER_NETWORK} -v %WORKSPACE%:/app -w /app python-chrome tail -f /dev/null"

                withCredentials([usernamePassword(credentialsId: 'log-collector-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                    // Clone the test framework repository
                    bat "docker exec ${TEST_CONTAINER} git clone -b dev https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/bienxhuy/devtest.git /app/devtest"
                }

                // Install Python dependencies
                bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pip install -r requirements.txt\""
            }
        }

        stage('Execute Test') {
            steps {
                bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pytest\""
            }
        }

        stage('Post Execution') {
            steps {
                bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && python ./postexec/main.py\""
            }
        }
    }

    post {
        always {
            echo 'Stopping services and cleaning up...'
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