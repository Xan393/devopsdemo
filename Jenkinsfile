pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'your-dockerhub-credentials-id' // Jenkins credentials ID for Docker Hub
        DOCKER_IMAGE_NAME = 'xandevops/xandevops1'
        REPO_URL = 'https://github.com/Xan393/devopsdemo.git'
        BRANCH_NAME = 'master' // Specify the branch you want to build
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    echo "Cloning repository from ${REPO_URL}..."
                    git branch: "${BRANCH_NAME}", url: "${REPO_URL}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile
                    echo "Building Docker image ${DOCKER_IMAGE_NAME}..."
                    def image = docker.build("${DOCKER_IMAGE_NAME}")
                    env.DOCKER_IMAGE_ID = image.id // Store the image ID for later use
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo "Running tests on Docker image ${DOCKER_IMAGE_NAME}..."
                    docker.image("${DOCKER_IMAGE_NAME}").inside {
                        sh 'your-test-command' // Replace with your actual test command
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker image ${DOCKER_IMAGE_NAME} to Docker Hub..."
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE_NAME}").push("latest")
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Cleaning up resources..."
                // Optionally, add cleanup commands if needed
            }
        }

        success {
            echo "Pipeline completed successfully!"
        }

        failure {
            echo "Pipeline failed. Please check the logs for details."
        }
    }
}
