pipeline {
    agent any

    stages {
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Fetch secret directly from AWS Secrets Manager (fresh every build)
                    def secretJson = sh(
                        script: "aws secretsmanager get-secret-value --secret-id docker-secret --region ap-south-1 --query SecretString --output text",
                        returnStdout: true
                    ).trim()

                    // Parse with jq (ensure jq is installed on the agent)
                    def dockerUser = sh(script: "echo '${secretJson}' | jq -r .username", returnStdout: true).trim()
                    def dockerPass = sh(script: "echo '${secretJson}' | jq -r .password", returnStdout: true).trim()

                    if (dockerUser == 'null' || dockerPass == 'null' || dockerUser == '' || dockerPass == '') {
                        error "Failed to parse Docker credentials from AWS secret – check JSON format/keys"
                    }

                    echo "Debug: Using Docker username = ${dockerUser}"

                    withEnv(["DOCKER_USER=${dockerUser}", "DOCKER_PASS=${dockerPass}"]) {
                        dir('src') {
                            sh '''
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin || {
                                    echo "Docker login failed - check AWS secret value / token validity"
                                    exit 1
                                }

                                docker build -t "$DOCKER_USER/checkoutservice:latest" .

                                docker push "$DOCKER_USER/checkoutservice:latest"
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            // Always clean up credentials from the agent
            sh 'docker logout || true'
        }
    }
}
