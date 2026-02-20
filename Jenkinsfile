pipeline {
    agent any

    stages {
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Fetch secret directly from AWS (fresh every run)
                    def secretJson = sh(
                        script: "aws secretsmanager get-secret-value --secret-id docker-secret --region ap-south-1 --query SecretString --output text",
                        returnStdout: true
                    ).trim()

                    def dockerUser = sh(script: "echo '${secretJson}' | jq -r .username", returnStdout: true).trim()
                    def dockerPass = sh(script: "echo '${secretJson}' | jq -r .password", returnStdout: true).trim()

                    if (dockerUser == 'null' || dockerPass == 'null' || dockerUser == '' || dockerPass == '') {
                        error "Failed to parse Docker credentials from AWS secret"
                    }

                    echo "Debug: Using Docker username = ${dockerUser}"

                    withEnv(["DOCKER_USER=${dockerUser}", "DOCKER_PASS=${dockerPass}"]) {
                        // No dir('src') — build from repo root
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin || {
                                echo "Docker login failed"
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

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
