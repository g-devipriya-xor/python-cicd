pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/g-devipriya-xor/python-cicd', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Use Minikube's Docker daemon
                sh '''
                    echo "Setting Docker environment for Minikube..."
                    eval $(minikube -p minikube docker-env)
                    docker build -t python-cicd:latest .
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying to Minikube..."
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Verifying pods..."
                    kubectl get pods
                    echo "Getting services..."
                    kubectl get svc
                '''
            }
        }
    }
}
