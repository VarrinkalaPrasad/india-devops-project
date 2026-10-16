
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }

    environment {
        APP_NAME = 'india-web'
        CONTAINER_NAME = 'india-web-ci-test'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm

                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=12 HEAD',
                        returnStdout: true
                    ).trim()

                    env.IMAGE_NAME = "${APP_NAME}:${IMAGE_TAG}"
                }

                echo "Building image: ${IMAGE_NAME}"
            }
        }

        stage('Validate Files') {
            steps {
                sh '''
                    set -eu

                    test -f Dockerfile
                    test -f src/index.html

                    echo "Dockerfile found"
                    echo "Application HTML found"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -eu

                    docker build \
                      --pull \
                      -t "$IMAGE_NAME" \
                      .

                    docker image inspect "$IMAGE_NAME" >/dev/null

                    echo "Docker image build successful"
                    docker images "$APP_NAME"
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    set -eu

                    docker rm -f "$CONTAINER_NAME" >/dev/null 2>&1 || true

                    docker run -d \
                      --name "$CONTAINER_NAME" \
                      "$IMAGE_NAME"

                    trap 'docker rm -f "$CONTAINER_NAME" >/dev/null 2>&1 || true' EXIT

                    for attempt in $(seq 1 15); do
                      if docker exec "$CONTAINER_NAME" \
                        wget -qO- http://127.0.0.1/ \
                        > /tmp/india-web-response.html 2>/dev/null; then
                        break
                      fi

                      sleep 2
                    done

                    grep -q "I LOVE INDIA" /tmp/india-web-response.html

                    echo "Smoke test passed: expected web page returned"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI PASSED: checkout, validation, image build and smoke test succeeded.'
        }

        failure {
            echo 'CI FAILED: inspect the first failed stage and its console output.'
        }

        always {
            sh '''
                docker rm -f "$CONTAINER_NAME" >/dev/null 2>&1 || true
                rm -f /tmp/india-web-response.html
            '''
        }
    }
}

