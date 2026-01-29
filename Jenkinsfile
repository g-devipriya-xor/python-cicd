pipeline {
    agent any

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

        stage('Build & Deploy Docker Image in Minikube') {
            steps {
                script {
                    // Use the Secret File for kubeconfig
                    withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                        echo "Using kubeconfig from Jenkins secret: ${KUBECONFIG}"

                        // Get short Git commit hash for unique image tag
                        IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                        IMAGE_NAME = "python-cicd:${IMAGE_TAG}"
                        echo "Building Docker image: ${IMAGE_NAME}"

                        // Build Docker image inside Minikube
                        sh """
                            # Point to Minikube's Docker daemon
                            eval \$(minikube -p minikube docker-env --shell bash)
                            
                            # Build Docker image
                            docker build -t ${IMAGE_NAME} .
                            
                            # Deploy to Kubernetes
                            kubectl apply -f k8s/deployment.yaml
                            kubectl set image deployment/python-cicd python-cicd=${IMAGE_NAME}
                        """
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo "Checking deployment rollout..."
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
