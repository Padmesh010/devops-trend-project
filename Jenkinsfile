pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "padmeshka/trend:latest"
        DOCKER_CREDS = "dockerhub-creds"
        KUBE_NAMESPACE = "trend"
    }

    stages {

        stage("Checkout Code") {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/Padmesh010/devops-react-app.git'
            }
        }

        stage("Build Docker Image") {
            steps {
                sh '''
                  docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage("Push Docker Image") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "$DOCKER_CREDS",
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                sh '''
                  kubectl apply -f ~/trend-k8s/deployment.yaml
                  kubectl apply -f ~/trend-k8s/service.yaml
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
