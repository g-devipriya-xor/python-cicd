pipeline {
    agent any

    environment {
        // Unique image tag using Git commit
        IMAGE_TAG = ""
        IMAGE_NAME = ""
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

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    IMAGE_NAME = "python-cicd:${IMAGE_TAG}"
                    echo "Building Docker image: ${IMAGE_NAME}"

                    sh """
                        docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Load Image into Minikube') {
            steps {
                script {
                    echo "Loading Docker image into Minikube..."
                    sh """
                        minikube image load ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                    script {
                        echo "Deploying to Minikube using kubeconfig secret..."
                        
                        sh """
                            export KUBECONFIG=${KUBECONFIG}

                            # Apply Kubernetes manifests
                            kubectl apply -f k8s/deployment.yaml

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
                        export KUBECONFIG=${KUBECONFIG}
                        echo "Checking deployment rollout..."
                        kubectl rollout status deployment/python-cicd
                        kubectl get pods -o wide
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}
