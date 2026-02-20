pipeline {
    agent any
    stages {
        stage('Test Credential') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-secret',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    // Use sh echo instead of Groovy echo to reduce masking confusion
                    sh 'echo "Username loaded: $DOCKER_USER"'
                }
            }
        }
    }
}
