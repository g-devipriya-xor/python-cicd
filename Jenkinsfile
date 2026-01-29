pipeline {
    agent any

    environment {
        // Jenkins secret will provide kubeconfig
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
                echo "Listing files in workspace..."
                sh 'ls -la'
            }
        }

        stage('Build Docker Image in Minikube') {
            steps {
                script {
                    def imageTag = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def imageName = "python-cicd:${imageTag}"
                    echo "Building Docker image: ${imageName}"

                    // Use Jenkins secret file for kubeconfig
                    withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                        sh """
                            wsl -d Ubuntu -e bash -c '
                            export KUBECONFIG=${KUBECONFIG}
                            minikube status || minikube start
                            eval \$(minikube -p minikube docker-env)
                            docker build -t ${imageName} .
                            '
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def imageTag = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def imageName = "python-cicd:${imageTag}"

                    withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                        sh """
                            export KUBECONFIG=${KUBECONFIG}
                            if [ -f k8s/deployment.yaml ]; then
                                kubectl apply -f k8s/deployment.yaml
                                kubectl set image deployment/python-cicd python-cicd=${imageName}
                            else
                                echo "❌ deployment.yaml not found!"
                                exit 1
                            fi
                        """
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                        sh """
                            export KUBECONFIG=${KUBECONFIG}
                            kubectl rollout status deployment/python-cicd
                            kubectl get pods -o wide
                        """
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
