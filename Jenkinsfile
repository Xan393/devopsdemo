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
                    sh 'sudo docker build -t $DOCKER_IMAGE_NAME .'
                    // Capture the image ID from the build output
                    def buildOutput = sh(script: 'docker build -t $DOCKER_IMAGE_NAME .', returnStdout: true).trim()
                    // Extract image ID from the output (assuming the output includes 'Successfully built <image_id>')
                    def imageId = buildOutput.split("\n").find { it.startsWith("Successfully built") }?.split()[-1]
                    env.DOCKER_IMAGE_ID = image.id // Store the image ID for later use
                    echo "Docker image ID: ${env.DOCKER_IMAGE_ID}"
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo "Running tests on Docker image ${DOCKER_IMAGE_NAME}..."
                    docker.image("${DOCKER_IMAGE_NAME}").inside {
                        sh 'npm install' // Replace with your actual test command
                        sh 'npm test' 
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
