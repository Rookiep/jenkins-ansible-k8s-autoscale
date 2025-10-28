pipeline {
    agent any

    environment {
        KUBECONFIG = '/root/.kube/config'
    }

    stages {

        stage('Initialize') {
            steps {
                echo '🚀 Starting Jenkins CI/CD pipeline for Ansible + K8s...'
                sh 'pwd && ls -la'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh '''
                    echo "Checking Docker daemon status..."
                    docker info > /dev/null 2>&1 || (echo "❌ Docker daemon not running!" && exit 1)

                    echo "Building image..."
                    docker build -t jenkins-ansible-k8s-autoscale:latest .
                '''
            }
        }

        stage('Push Docker Image (Optional)') {
            when {
                expression { return false } // Enable if pushing to DockerHub or ECR
            }
            steps {
                echo '📦 Pushing Docker image to registry...'
                sh '''
                    docker tag jenkins-ansible-k8s-autoscale:latest your_dockerhub_username/jenkins-ansible-k8s-autoscale:latest
                    docker push your_dockerhub_username/jenkins-ansible-k8s-autoscale:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🚀 Deploying application to Kubernetes cluster...'
                sh '''
                    kubectl get nodes
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Run Ansible Node Health Check') {
            steps {
                echo '🧩 Running Ansible node health check...'
                sh '''
                    ansible --version
                    ansible-playbook ansible/node_health_check.yml
                '''
            }
        }

        stage('Apply Horizontal Pod Autoscaler (HPA)') {
            steps {
                echo '⚙️ Applying Horizontal Pod Autoscaler...'
                sh '''
                    kubectl apply -f k8s/hpa.yaml
                    kubectl get hpa
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment and health check completed successfully.'
        }
        failure {
            echo '❌ Build failed — check logs for details.'
        }
    }
}
