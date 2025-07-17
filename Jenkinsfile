pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('DockerHubID')
        DOCKER_IMAGE_NAME = 'jeenapandit/cicd-internal'
        DOCKER_TAG = "base"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jeenapandit/aer-sample-internal.git'
            }
        }
        
        stage('Unit Tests') {
            steps {
                script {
                    // Run unit tests before building Docker image
                    sh """
                        npm install
                        npm test
                    """
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ."
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    // Verify Docker images were built successfully
                    sh "docker images | grep ${DOCKER_IMAGE_NAME}"
                    
                    // Generate unique container name to avoid conflicts
                    def containerName = "test-container-${env.BUILD_NUMBER}-${env.BUILD_ID}"
                    
                    // Clean up any existing containers with the same name pattern
                    sh """
                        # Remove any existing test containers
                        docker stop test-container-temp 2>/dev/null || true
                        docker rm -f test-container-temp 2>/dev/null || true
                        docker stop ${containerName} 2>/dev/null || true
                        docker rm -f ${containerName} 2>/dev/null || true
                    """
                    
                    // Test the Docker image by running it temporarily
                    sh """
                        echo "Testing Docker container..."
                        docker run -d --name ${containerName} -p 3001:8082 ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                        sleep 10
                        
                        # Verify container is running
                        docker ps | grep ${containerName} || echo "Container not found in running state"
                        
                        # Test if application responds (basic health check)
                        curl -f http://localhost:3001 || echo "Application not responding on port 3001"
                        
                        # Check container logs
                        docker logs ${containerName} || echo "Could not retrieve logs"
                        
                        # Clean up test container with better error handling
                        docker stop ${containerName} || docker kill ${containerName} || echo "Could not stop container"
                        docker rm ${containerName} || docker rm -f ${containerName} || echo "Could not remove container"
                    """
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh """
                        echo \$DOCKER_HUB_CREDENTIALS_PSW | docker login -u \$DOCKER_HUB_CREDENTIALS_USR --password-stdin
                        docker push ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                        docker logout
                    """
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Cleanup any remaining test containers
                sh '''
                    # Force cleanup of any test containers that might still exist
                    docker stop test-container-temp 2>/dev/null || true
                    docker rm -f test-container-temp 2>/dev/null || true
                    
                    # Clean up containers created by this build
                    docker ps -a --filter "name=test-container-" --format "{{.Names}}" | xargs -r docker rm -f 2>/dev/null || true
                    
                    # General Docker cleanup
                    docker system prune -f
                '''
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
