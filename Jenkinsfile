pipeline {
    agent any

    environment {
        CLUSTER_NAME = 'ecommerce-cluster'
        REGION       = 'ap-south-1'
        NAMESPACE    = 'webapps'
        KUBECONFIG   = "${WORKSPACE}/kubeconfig"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Authenticate to EKS') {
            steps {
                sh '''
                  aws eks update-kubeconfig \
                    --name ${CLUSTER_NAME} \
                    --region ${REGION} \
                    --kubeconfig ${KUBECONFIG}
                '''
            }
        }

        stage('Verify Cluster') {
            steps {
                sh '''
                  kubectl --kubeconfig=${KUBECONFIG} get nodes
                  kubectl --kubeconfig=${KUBECONFIG} get ns
                '''
            }
        }

        stage('Deploy Microservices') {
            steps {
                sh '''
                  kubectl --kubeconfig=${KUBECONFIG} create namespace ${NAMESPACE} \
                    --dry-run=client -o yaml | kubectl apply -f -

                  kubectl --kubeconfig=${KUBECONFIG} apply -f deployment-service.yml -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                  kubectl --kubeconfig=${KUBECONFIG} get pods -n ${NAMESPACE}
                  kubectl --kubeconfig=${KUBECONFIG} get svc -n ${NAMESPACE}
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -f ${KUBECONFIG} || true'
        }
    }
}
