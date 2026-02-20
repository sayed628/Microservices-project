pipeline {
    agent any

    stages {
        stage('Build & Push Docker Image') {
            steps {
                dir('src') {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-secret',          // ← This matches exactly what you see in Jenkins
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            # Login (password is masked in console output)
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin || {
                                echo "Docker login failed – check if token is valid in AWS Secrets Manager"
                                exit 1
                            }

                            # Build
                            docker build -t "$DOCKER_USER/shippingservice:latest" .

                            # Push
                            docker push "$DOCKER_USER/shippingservice:latest"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up login session (good practice)
            sh 'docker logout || true'
        }
    }
}
