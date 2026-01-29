pipeline {
    agent any

    environment {
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

        stage('Verify Workspace') {
            steps {
                echo "Listing files in workspace..."
                sh 'ls -la'
            }
        }

        stage('Build Docker Image in Minikube') {
            steps {
                script {
                    // Get short Git commit hash for unique image tag
                    def IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def IMAGE_NAME = "python-cicd:${IMAGE_TAG}"
                    echo "Using image: ${IMAGE_NAME}"

                    sh """
                        # Point Docker to existing Minikube Docker daemon
                        eval \$(minikube -p minikube docker-env)

                        # Build Docker image in Minikube
                        docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes..."
                    sh """
                        # Check if manifest exists
                        if [ -f deployment.yaml ]; then
                            kubectl apply -f deployment.yaml
                        else
                            echo "❌ deployment.yaml not found!"
                            exit 1
                        fi

                        # Update deployment with the newly built image
                        kubectl set image deployment/python-cicd python-cicd=${IMAGE_NAME}
                    """
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
