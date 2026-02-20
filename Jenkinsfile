pipeline {
    agent any

    stages {

        stage('Fetch Docker Credentials') {
            steps {
                script {
                    def secret = sh(
                        script: "aws secretsmanager get-secret-value --secret-id docker-secret --query SecretString --output text",
                        returnStdout: true
                    ).trim()

                    def dockerUser = sh(
                        script: "echo '${secret}' | jq -r .username",
                        returnStdout: true
                    ).trim()

                    def dockerToken = sh(
                        script: "echo '${secret}' | jq -r .token",
                        returnStdout: true
                    ).trim()

                    env.DOCKER_USERNAME = dockerUser
                    env.DOCKER_TOKEN = dockerToken
                }
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                dir('src') {
                    sh """
                      echo \$DOCKER_TOKEN | docker login -u \$DOCKER_USERNAME --password-stdin
                      docker build -t \$DOCKER_USERNAME/shippingservice:latest .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('src') {
                    sh "docker push \$DOCKER_USERNAME/shippingservice:latest"
                }
            }
        }
    }
}
