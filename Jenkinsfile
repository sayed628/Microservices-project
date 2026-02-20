pipeline {
    agent any

    stages {
        stage('Build & Push Docker Image') {
            steps {
                dir('src') {
                    script {
                        // Fetch latest secret JSON directly from AWS
                        def secretJson = sh(
                            script: "aws secretsmanager get-secret-value --secret-id docker-secret --region ap-south-1 --query SecretString --output text",
                            returnStdout: true
                        ).trim()

                        // Parse (jq must be installed on EC2 agent)
                        def dockerUser = sh(script: "echo '${secretJson}' | jq -r .username", returnStdout: true).trim()
                        def dockerPass = sh(script: "echo '${secretJson}' | jq -r .password", returnStdout: true).trim()

                        // Debug print (username is safe to echo)
                        echo "Debug: Loaded Docker username from AWS secret = ${dockerUser}"

                        if (dockerUser == 'null' || dockerPass == 'null' || dockerUser == '' || dockerPass == '') {
                            error "Failed to parse Docker credentials - check secret JSON in AWS Secrets Manager"
                        }

                        withEnv(["DOCKER_USER=${dockerUser}", "DOCKER_PASS=${dockerPass}"]) {
                            sh '''
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin || {
                                    echo "Docker login failed - verify token is valid (manual test worked, so check if secret updated correctly)"
                                    exit 1
                                }
                                docker build -t "$DOCKER_USER/shippingservice:latest" .
                                docker push "$DOCKER_USER/shippingservice:latest"
                            '''
                        }
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
