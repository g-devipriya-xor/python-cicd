pipeline {
    agent any

    environment {
        // Kubernetes config from Jenkins secret
        KUBECONFIG = ''
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
                echo "Listing workspace files..."
                sh 'ls -la'
            }
        }

        stage('Build Docker Image in Minikube') {
            steps {
                script {
                    def IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def IMAGE_NAME = "python-cicd:${IMAGE_TAG}"
                    echo "Building Docker image: ${IMAGE_NAME}"

                    // Use kubeconfig secret only for kubectl
                    withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                        sh """
                            # Unset any previous Docker TLS envs
                            eval \$(minikube -p minikube docker-env --unset)

                            # Get Minikube IP and use TCP without TLS
                            MINIKUBE_IP=\$(minikube ip)
                            export DOCKER_HOST="tcp://\$MINIKUBE_IP:2375"
                            export DOCKER_TLS_VERIFY=""

                            # Build the Docker image
                            docker build -t ${IMAGE_NAME} .
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def IMAGE_NAME = "python-cicd:${IMAGE_TAG}"

                    withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                        sh """
                            # Apply deployment manifest
                            if [ -f k8s/deployment.yaml ]; then
                                kubectl apply -f k8s/deployment.yaml
                            else
                                echo "❌ k8s/deployment.yaml not found!"
                                exit 1
                            fi

                            # Update image in deployment
                            kubectl set image deployment/python-cicd python-cicd=${IMAGE_NAME}
                        """
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                    sh """
                        echo "Checking rollout status..."
                        kubectl rollout status deployment/python-cicd
                        kubectl get pods -o wide
                    """
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
