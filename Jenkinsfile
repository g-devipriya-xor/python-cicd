pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-cicd:latest"
        K8S_DEPLOYMENT_NAME = "python-cicd"
        K8S_NAMESPACE = "default"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image in Minikube') {
            steps {
                script {
                    echo "Configuring Docker to use Minikube daemon..."
                    sh '''
                        # Point Docker to Minikube's Docker daemon
                        eval $(minikube -p minikube docker-env)
                        # Build the image inside Minikube
                        docker build -t $IMAGE_NAME .
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Applying Kubernetes deployment..."
                    sh '''
                        # Apply deployment.yaml
                        kubectl apply -f deployment.yaml -n $K8S_NAMESPACE

                        # Force the deployment to use the newly built image
                        kubectl set image deployment/$K8S_DEPLOYMENT_NAME \
                        $K8S_DEPLOYMENT_NAME=$IMAGE_NAME -n $K8S_NAMESPACE
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh '''
                        echo "Checking pods in Kubernetes..."
                        kubectl get pods -n $K8S_NAMESPACE
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! ✅"
        }
        failure {
            echo "Pipeline failed. Check logs for details. ❌"
        }
    }
}
