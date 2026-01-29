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

        stage('Verify Workspace') {
            steps {
                echo "Listing files in workspace..."
                sh 'ls -la'
                sh 'ls -la k8s'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Get short Git commit hash for unique image tag
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
                echo "Loading Docker image into Minikube..."
                sh """
                    minikube image load ${IMAGE_NAME}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Minikube using kubeconfig secret..."
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBE_SECRET')]) {
                    sh """
                        TMP_DIR=\$(mktemp -d)
                        unzip -o \$KUBE_SECRET -d \$TMP_DIR

                        export KUBECONFIG=\$TMP_DIR/minikube-config.yaml

                        # Apply deployment (relative path to workspace root)
                        kubectl apply -f k8s/deployment.yaml

                        # Update deployment image
                        kubectl set image deployment/python-cicd python-cicd=${IMAGE_NAME} -n default

                        # Wait for rollout
                        kubectl rollout status deployment/python-cicd -n default
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Verifying deployment..."
                sh """
                    kubectl get pods -o wide
                    kubectl get svc -o wide
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}
