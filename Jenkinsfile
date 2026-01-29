pipeline {
    agent any

    environment {
        // Use the Kubernetes Secret File credential with your minikube-config.yaml
        KUBECONFIG = credentials('minikube-config')
    }

    stages {

        stage('Checkout SCM') {
            steps {
                echo "Checking out Git repository..."
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/g-devipriya-xor/python-cicd',
                        credentialsId: 'c1e07a47-7c93-460f-ae7b-0514f937d43b'
                    ]]
                ])
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
                script {
                    echo "Deploying to Minikube..."

                    sh """
                        kubectl apply -f k8s/deployment.yaml
                        kubectl set image deployment/python-cicd python-cicd=${IMAGE_NAME}
                    """
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
