pipeline {
    agent any

    environment {
        DOCKER_SECRET = sh(script: "aws secretsmanager get-secret-value --secret-id docker-secret --query SecretString --output text", returnStdout: true).trim()
        DOCKER_USERNAME = sh(script: "echo $DOCKER_SECRET | jq -r .username", returnStdout: true).trim()
        DOCKER_PASSWORD = sh(script: "echo $DOCKER_SECRET | jq -r .password", returnStdout: true).trim()
    }

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    sh "docker build -t sayed/adservice:latest ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push sayed/adservice:latest"
                }
            }
        }
    }
}
