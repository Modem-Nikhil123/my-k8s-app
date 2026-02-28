pipeline {
    agent any

    stages {

         stage('Checkout Code') {
            steps {
                echo 'Code already checked out from Git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t my-k8s-app:${BUILD_NUMBER} .
                docker tag my-k8s-app:${BUILD_NUMBER} nikhil0418/my-k8s-app:latest
                '''
            }
        }

        stage('Push Docker Image') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'docker-hub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push nikhil0418/my-k8s-app:latest
            '''
        }
        }
    }

        stage('Start Minikube if not running') {
            steps {
                sh '''
                if ! minikube status | grep -q "apiserver: Running"; then
                    echo "Minikube is not running. Starting now..."
                    minikube start --driver=docker --memory=2048 --cpus=2
                fi
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # Load latest image into Minikube
                # minikube image load nikhil0418/my-k8s-app:latest

                # Apply manifests
                minikube kubectl -- apply -f k8s/deployment.yaml
                minikube kubectl -- apply -f k8s/service.yaml
                minikube service my-k8s-app-service
                '''
            }
        }
    }
}