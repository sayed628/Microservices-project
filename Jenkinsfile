pipeline {
    agent any

    stages {
        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-secret',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    dir('src') {
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
