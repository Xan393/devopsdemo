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
            echo "Building Docker image ${DOCKER_IMAGE_NAME}..."
            
            // Ensure Docker is executable without 'sudo'
            def buildOutput = sh(script: "docker build -t ${DOCKER_IMAGE_NAME} .", returnStdout: true).trim()
            
            // Extract the image ID
            def imageId = buildOutput.split("\n").find { it.contains("Successfully built") }
            if (imageId) {
                imageId = imageId.split()[-1] // Extract the image ID
                env.DOCKER_IMAGE_ID = imageId // Store image ID in the environment variable
                echo "Docker image ID: ${env.DOCKER_IMAGE_ID}"
            } else {
                error "Docker build failed, unable to extract image ID"
            }
        }
    }
}

stage('Run Tests') {
    steps {
        script {
            echo "Running tests on Docker image ${DOCKER_IMAGE_NAME}..."

            // In case Jenkins is not inside a Docker container, use the host environment directly
            // Check if $HOME is set
            sh 'echo "HOME: $HOME"'
            
            // If $HOME is not set, set it to the default Jenkins home directory
            sh '''
                if [ -z "$HOME" ]; then
                    echo "HOME is not set. Using default Jenkins home directory /var/lib/jenkins";
                    export HOME=/var/lib/jenkins;
                else
                    echo "Using HOME: $HOME";
                fi
            '''

            // Clear npm cache and fix permissions
            sh 'npm cache clean --force'

            // Ensure the npm directory exists and has the correct permissions
            sh 'mkdir -p $HOME/.npm'  // Ensure the ~/.npm directory exists
            sh 'chown -R $(id -u):$(id -g) $HOME/.npm'  // Fix ownership

            // Install dependencies
            sh 'npm install'

            // Fix permissions of node_modules directory (after npm install)
            sh 'chown -R $(id -u):$(id -g) node_modules'

            // Run tests
            sh 'npm test'
        }
    }
}


            
        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker image ${DOCKER_IMAGE_NAME} to Docker Hub..."
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
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
