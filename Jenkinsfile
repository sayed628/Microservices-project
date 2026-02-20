pipeline {
    agent any

    stages {
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Fetch latest secret directly from AWS Secrets Manager
                    def secretJson = sh(
                        script: "aws secretsmanager get-secret-value --secret-id docker-secret --region ap-south-1 --query SecretString --output text",
                        returnStdout: true
                    ).trim()

                    // Parse with jq (make sure jq is installed on the agent)
                    def dockerUser = sh(script: "echo '${secretJson}' | jq -r .username", returnStdout: true).trim()
                    def dockerPass = sh(script: "echo '${secretJson}' | jq -r .password", returnStdout: true).trim()

                    if (dockerUser == 'null' || dockerPass == 'null' || dockerUser == '' || dockerPass == '') {
                        error "Failed to parse Docker credentials from AWS secret - check JSON format"
                    }

                    echo "Debug: Using Docker username = ${dockerUser}"

                    withEnv(["DOCKER_USER=${dockerUser}", "DOCKER_PASS=${dockerPass}"]) {
                        dir('src') {
                            sh '''
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin || {
                                    echo "Docker login failed - verify token in AWS Secrets Manager"
                                    exit 1
                                }

                                docker build -t "$DOCKER_USER/cartservice:latest" .

                                docker push "$DOCKER_USER/cartservice:latest"
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up login credentials
            sh 'docker logout || true'
        }
    }
}
