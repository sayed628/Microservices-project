pipeline {
    agent any

    environment {
        # Fetch Docker credentials from AWS Secrets Manager
        DOCKER_SECRET = sh(script: "aws secretsmanager get-secret-value --secret-id docker-secret --query SecretString --output text", returnStdout: true).trim()
        DOCKER_USERNAME = sh(script: "echo $DOCKER_SECRET | jq -r .username", returnStdout: true).trim()
        DOCKER_PASSWORD = sh(script: "echo $DOCKER_SECRET | jq -r .password", returnStdout: true).trim()
    }

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    dir('src') {
                        // Login to Docker Hub using AWS Secrets Manager credentials
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                        // Build image
                        sh "docker build -t $DOCKER_USERNAME/checkoutservice:latest ."
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    dir('src') {
                        // Push image to Docker Hub
                        sh "docker push $DOCKER_USERNAME/checkoutservice:latest"
                    }
                }
            }
        }
    }
}
