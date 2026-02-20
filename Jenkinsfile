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
                    echo "Username from credential: ${DOCKER_USER}"
                    // Don't echo password!
                }
            }
        }
    }
}
