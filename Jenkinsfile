pipeline {
    agent any

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                dir('src') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker build -t "$DOCKER_USER/shippingservice:latest" .
                            docker push "$DOCKER_USER/shippingservice:latest"
                        '''
                    }
                }
            }
        }
    }
}
