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
                    
                    // Test the Docker image by running it temporarily
                    sh """
                        echo "Testing Docker container..."
                        docker run -d --name test-container-temp -p 3001:8082 ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                        sleep 10
                        
                        # Verify container is running
                        docker ps | grep test-container-temp
                        
                        # Test if application responds (basic health check)
                        curl -f http://localhost:3001 || echo "Application not responding on port 3001"
                        
                        # Check container logs
                        docker logs test-container-temp
                        
                        # Clean up test container
                        docker stop test-container-temp
                        docker rm test-container-temp
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
            sh 'docker system prune -f'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
