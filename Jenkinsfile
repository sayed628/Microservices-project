pipeline {
    agent any

    stages {
        stage('Fetch Docker Credentials') {
            steps {
                script {
                    DOCKER_SECRET = sh(
                        script: "aws secretsmanager get-secret-value --secret-id docker-secret --query SecretString --output text",
                        returnStdout: true
                    ).trim()

                    DOCKER_USERNAME = sh(
                        script: "echo '${DOCKER_SECRET}' | jq -r .username",
                        returnStdout: true
                    ).trim()

                    DOCKER_PASSWORD = sh(
                        script: "echo '${DOCKER_SECRET}' | jq -r .password",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                dir('src') {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    sh "docker build -t ${DOCKER_USERNAME}/shippingservice:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('src') {
                    sh "docker push ${DOCKER_USERNAME}/shippingservice:latest"
                }
            }
        }
    }
}
