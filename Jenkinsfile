pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "padmeshka/trend:latest"
        DOCKERHUB_CREDS = "dockerhub-creds"
        GITHUB_CREDS = "github-creds"
        KUBE_NAMESPACE = "trend"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/Padmesh010/devops-trend-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image..."
                docker build -t padmeshka/trend:latest -f docker/Dockerfile .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                echo "Deploying to EKS..."

                kubectl apply -f trend-k8s/deployment.yaml
                kubectl apply -f trend-k8s/service.yaml

                kubectl rollout status deployment/trend-app -n trend
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "CI/CD Pipeline failed!"
        }
    }
}

