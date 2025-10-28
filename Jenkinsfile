pipeline {
    agent any

    environment {
        KUBECONFIG = '/root/.kube/config'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Checking out source code from main branch...'
                git branch: 'main', url: 'https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh '''
                    docker build -t jenkins-ansible-k8s-autoscale:latest .
                '''
            }
        }

        stage('Push Docker Image (Optional)') {
            when {
                expression { return false } // disable if no Docker registry
            }
            steps {
                echo '📦 Pushing Docker image to registry...'
                sh '''
                    docker tag jenkins-ansible-k8s-autoscale:latest your_dockerhub_username/jenkins-ansible-k8s-autoscale:latest
                    docker push your_dockerhub_username/jenkins-ansible-k8s-autoscale:latest
                '''
            }
        }

        stage('Deploy to K8s') {
            steps {
                echo '🚀 Deploying application to Kubernetes...'
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Run Ansible Node Health Check') {
            steps {
                echo '🧩 Running Ansible health check on Kubernetes nodes...'
                sh '''
                    ansible-playbook ansible/node_health_check.yml
                '''
            }
        }

        stage('Apply HPA (Horizontal Pod Autoscaler)') {
            steps {
                echo '⚙️ Applying Horizontal Pod Autoscaler...'
                sh '''
                    kubectl apply -f k8s/hpa.yaml
                '''
            }
        }

    }

    post {
        success {
            echo '✅ Deployment and health check completed successfully.'
        }
        failure {
            echo '❌ Build failed — please check logs for details.'
        }
    }
}

