pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/g-devipriya-xor/python-cicd',
                        credentialsId: 'c1e07a47-7c93-460f-ae7b-0514f937d43b' // Git credentials
                    ]]
                ])
            }
        }

        stage('Verify Workspace') {
            steps {
                echo "Listing files in workspace..."
                sh 'ls -la'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Use the secret kubeconfig file stored in Jenkins
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                    script {
                        echo "Using kubeconfig from Jenkins secret: ${KUBECONFIG}"
                        sh '''
                            # Check cluster connectivity
                            kubectl cluster-info
                            
                            # Apply Kubernetes manifests
                            kubectl apply -f k8s/deployment.yaml
                            
                            # Optional: update image if using CI-built image
                            # kubectl set image deployment/python-cicd python-cicd=python-cicd:latest
                            
                            # Verify rollout
                            kubectl rollout status deployment/python-cicd
                            kubectl get pods -o wide
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs above."
        }
    }
}
