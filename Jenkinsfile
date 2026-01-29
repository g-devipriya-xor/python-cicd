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
                    extensions: [[$class: 'CleanBeforeCheckout']], // ensures workspace is clean
                    userRemoteConfigs: [[
                        url: 'https://github.com/g-devipriya-xor/python-cicd',
                        credentialsId: 'c1e07a47-7c93-460f-ae7b-0514f937d43b'
                    ]]
                ])
            }
        }

        stage('Verify Workspace') {
            steps {
                echo "Listing workspace root..."
                sh 'ls -la'
                echo "Listing k8s folder..."
                sh 'ls -la k8s'
            }
        }

        stage('Build Docker Image in Minikube') {
            steps {
                script {
                    IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    IMAGE_NAME = "python-cicd:${IMAGE_TAG}"
                    echo "Building Docker image directly in Minikube: ${IMAGE_NAME}"

                    // Point Docker CLI to Minikube's Docker daemon
                    sh """
                        eval \$(minikube -p minikube docker-env)
                        docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Minikube using kubeconfig secret..."
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBE_SECRET')]) {
                    script {
                        sh """
                            TMP_DIR=\$(mktemp -d)
                            unzip -o \$KUBE_SECRET -d \$TMP_DIR
                            export KUBECONFIG=\$TMP_DIR/kubeconfigs/minikube-config.yaml

                            # Apply deployment (initial create or update)
                            kubectl apply -f k8s/deployment.yaml

                            # Update deployment to use dynamically tagged image
                            kubectl set image deployment/python-cicd python-cicd=${IMAGE_NAME} -n default

                            # Wait for rollout to finish
                            kubectl rollout status deployment/python-cicd -n default
                        """
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Verifying deployment..."
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBE_SECRET')]) {
                    sh """
                        TMP_DIR=\$(mktemp -d)
                        unzip -o \$KUBE_SECRET -d \$TMP_DIR
                        export KUBECONFIG=\$TMP_DIR/kubeconfigs/minikube-config.yaml

                        kubectl get pods -o wide
                        kubectl get svc -o wide
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
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}
