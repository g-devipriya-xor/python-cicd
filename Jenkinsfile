pipeline {
    agent any

    environment {
        // Point to Jenkins user’s kube config
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/g-devipriya-xor/python-cicd',
                        credentialsId: 'c1e07a47-7c93-460f-ae7b-0514f937d43b'
                    ]]
                ])
            }
        }

        stage('Build Docker Image in Minikube') {
            steps {
                script {
                    echo "Building Docker image directly in Minikube..."
                    sh '''
                        # Ensure Minikube is running
                        minikube status || minikube start
                        
                        # Build image inside Minikube without TLS issues
                        minikube -p minikube image build -t python-cicd:latest .
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes..."
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl set image deployment/python-cicd python-cicd=python-cicd:latest
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo "Checking rollout status..."
                    sh 'kubectl rollout status deployment/python-cicd'
                    sh 'kubectl get pods -o wide'
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
