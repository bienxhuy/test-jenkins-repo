pipeline {
    agent any

    environment {
        // Use existing Docker Compose network
        DOCKER_NETWORK = "${env.DOCKER_NETWORK ?: 'test_default'}"
        // Define container names
        DEV_CONTAINER = 'dev-instance'
        DEV_PORT = "${env.DEV_PORT ?: '4173'}"
        TEST_CONTAINER = 'test-instance'
        // Define BASE_URL for test framework (using container name for communication within Docker network)
        BASE_URL = "http://${DEV_CONTAINER}:${DEV_PORT}"
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
                    bat "docker run -d --name ${DEV_CONTAINER} --network ${DOCKER_NETWORK} -p ${DEV_PORT}:${DEV_PORT} -v %WORKSPACE%:/app -w /app node:22.16.0 tail -f /dev/null"

                    // Clone the repository inside the container
                    bat "docker exec ${DEV_CONTAINER} git clone -b master https://github.com/bienxhuy/test-instance.git /app/test-instance"

                    // Install dependencies
                    bat "docker exec ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm install\""

                    // Build the application
                    bat "docker exec ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm run build\""

                    // Start the server in the background
                    bat "docker exec -d ${DEV_CONTAINER} bash -c \"cd /app/test-instance && npm run preview -- --host 0.0.0.0\""

                    sleep time: 60, unit: 'SECONDS'
                }
            }
        }

        // stage('Prepare Test Framework') {
        //     steps {
        //         script {
        //             withCredentials([usernamePassword(credentialsId: 'log-collector-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
        //                 // Pull Python Docker image (specific version)
        //                 bat 'docker pull python:3.12.10'

        //                 // Run Python container in the existing network with BASE_URL environment variable
        //                 bat "docker run -d --name ${TEST_CONTAINER} --network ${DOCKER_NETWORK} -v %WORKSPACE%:/app -w /app -e BASE_URL=${BASE_URL} python:3.12.10 tail -f /dev/null"

        //                 // Install system dependencies, Google Chrome, and ChromeDriver
        //                 bat """
        //                     docker exec ${TEST_CONTAINER} bash -c ^
        //                     "apt-get update && ^
        //                     apt-get install -y wget unzip gnupg2 && ^
        //                     wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - && ^
        //                     echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' >> /etc/apt/sources.list.d/google-chrome.list && ^
        //                     apt-get update && ^
        //                     apt-get install -y google-chrome-stable && ^
        //                     CHROME_VERSION=\\$(google-chrome --version | grep -oE '[0-9]+\\.[0-9]+\\.[0-9]+') && ^
        //                     wget -O /tmp/chromedriver.zip https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/\\${CHROME_VERSION}/linux64/chromedriver-linux64.zip && ^
        //                     unzip /tmp/chromedriver.zip -d /usr/local/bin/ && ^
        //                     mv /usr/local/bin/chromedriver-linux64/chromedriver /usr/local/bin/chromedriver && ^
        //                     chmod +x /usr/local/bin/chromedriver && ^
        //                     rm /tmp/chromedriver.zip"
        //                 """

        //                 // Clone the test framework repository inside the container
        //                 bat "docker exec ${TEST_CONTAINER} git clone -b dev https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/bienxhuy/devtest.git /app/devtest"

        //                 // Install Python dependencies
        //                 bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pip install -r requirements.txt\""
        //             }
        //         }
        //     }
        // }

        // stage('Execute Test') {
        //     steps {
        //         script {
        //             echo 'Executing tests...'
        //             // Run pytest in the test framework directory
        //             bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pytest\""
        //             // Note: Assumes pytest is configured to send results to InfluxDB at influxdb3:8181
        //             // If additional arguments are needed, e.g., --influxdb-host=${INFLUXDB_HOST} --influxdb-port=${INFLUXDB_PORT}, add them here
        //         }
        //     }
        // }

        // stage('Data Manipulation') {
        //     steps {
        //         echo 'Collecting test results...'
        //         // Add steps to collect and process test results from the test instance
        //         // Example: Copy results from test container to workspace
        //         // bat "docker cp ${TEST_CONTAINER}:/app/devtest/results %WORKSPACE%/results"
        //     }
        // }
    }

    post {
        always {
            echo 'Stopping services and cleaning up...'
            script {
                // Stop and remove only the dev and test containers
                bat "docker stop ${DEV_CONTAINER} || echo 'Error stopping ${DEV_CONTAINER}'"
                bat "docker rm ${DEV_CONTAINER} || echo 'Error removing ${DEV_CONTAINER}'"
                // bat "docker stop ${TEST_CONTAINER} || echo 'Error stopping ${TEST_CONTAINER}'"
                // bat "docker rm ${TEST_CONTAINER} || echo 'Error removing ${TEST_CONTAINER}'"
                // Do not remove the existing Docker network or InfluxDB/Grafana containers
                // Clean Jenkins workspace
                cleanWs()
            }
        }
    }
}