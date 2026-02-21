pipeline {
    agent any

    environment {
        CLUSTER_NAME = 'ecom-eks'         // ← your new cluster name
        REGION       = 'ap-south-1'
        NAMESPACE    = 'webapps'
    }

    stages {
        stage('Authenticate to EKS') {
            steps {
                script {
                    // Use EC2 IAM role to generate fresh kubeconfig (no hardcoded creds)
                    sh """
                        aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${REGION} --kubeconfig kubeconfig
                        export KUBECONFIG=\$(pwd)/kubeconfig
                        kubectl config use-context arn:aws:eks:${REGION}:365202716362:cluster/${CLUSTER_NAME}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                        kubectl apply -f deployment-service.yml -n ${NAMESPACE}
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh """
                        echo "Waiting for pods to be ready..."
                        kubectl wait --for=condition=Ready pod -l app=frontend -n ${NAMESPACE} --timeout=300s || echo "Timeout - check logs"
                        
                        echo "Pods:"
                        kubectl get pods -n ${NAMESPACE} -o wide
                        
                        echo "Services:"
                        kubectl get svc -n ${NAMESPACE}
                        
                        echo "Ingress (if any):"
                        kubectl get ingress -n ${NAMESPACE} || true
                    """
                }
            }
        }

        // Optional: Cleanup stage (uncomment when you want to tear down)
        // stage('Cleanup') {
        //     steps {
        //         sh "kubectl delete -f deployment-service.yml -n ${NAMESPACE} --ignore-not-found=true"
        //     }
        // }
    }

    post {
        always {
            // Clean up temporary kubeconfig
            sh 'rm -f kubeconfig || true'
        }
    }
}
