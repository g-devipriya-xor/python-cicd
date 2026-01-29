pipeline {
    agent any

    environment {
        IMAGE_NAME = ""
        IMAGE_TAG = ""
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

        stage('Build Docker Image Locally') {
            steps {
                script {
                    echo "Building Docker image locally..."
                    // Get short git commit hash for tag
                    IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    IMAGE_NAME = "python-cicd:${IMAGE_TAG}"
                    echo "Image name: ${IMAGE_NAME}"

                    sh """
                        docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Load Image into Minikube') {
            steps {
                script {
                    echo "Loading image into Minikube..."
                    sh """
                        minikube image load ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBE_SECRET')]) {
                    script {
                        echo "Deploying to Minikube using kubeconfig secret..."
                        sh """
                            # Create temporary folder and unzip kubeconfig files
                            TMP_DIR=\$(mktemp -d)
                            unzip -o \$KUBE_SECRET -d \$TMP_DIR

                            export KUBECONFIG=\$TMP_DIR/minikube-config.yaml

                            # Apply deployment
                            kubectl apply -f k8s/deployment.yaml

                            # Update deployment image
                            kubectl set image deployment/python-cicd python-cicd=${IMAGE_NAME}

                            # Clean up
                            rm -rf \$TMP_DIR
                        """
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo "Checking rollout status..."
                    sh """
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
