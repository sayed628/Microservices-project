pipeline {
    agent any

    stages {
        stage('Build & Push Docker Image') {
            steps {
                // No dir('src') — build from current workspace root
                script {
                    def secretJson = sh(
                        script: "aws secretsmanager get-secret-value --secret-id docker-secret --region ap-south-1 --query SecretString --output text",
                        returnStdout: true
                    ).trim()

                    def dockerUser = sh(script: "echo '${secretJson}' | jq -r .username", returnStdout: true).trim()
                    def dockerPass = sh(script: "echo '${secretJson}' | jq -r .password", returnStdout: true).trim()

                    echo "Debug: Loaded Docker username from AWS secret = ${dockerUser}"

                    withEnv(["DOCKER_USER=${dockerUser}", "DOCKER_PASS=${dockerPass}"]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin || exit 1
                            docker build -t "$DOCKER_USER/shippingservice:latest" .
                            docker push "$DOCKER_USER/shippingservice:latest"
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
