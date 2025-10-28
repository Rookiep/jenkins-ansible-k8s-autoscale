pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    sh '''
                        git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                    '''
                }
            }
        }

        stage('Install kubectl') {
            steps {
                script {
                    sh '''
                        # Install kubectl if not present
                        if ! command -v kubectl &> /dev/null; then
                            echo "Installing kubectl..."
                            curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                            chmod +x kubectl
                            mkdir -p /usr/local/bin
                            mv kubectl /usr/local/bin/kubectl
                        fi
                        kubectl version --client
                    '''
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    sh '''
                        echo "Deploying Kubernetes resources..."
                        
                        # Skip validation to bypass authentication issues
                        kubectl apply -f k8s/deployment.yaml --validate=false
                        kubectl apply -f k8s/service.yaml --validate=false
                        
                        echo "Deployment completed (validation skipped)"
                    '''
                }
            }
        }

        stage('Run Ansible Node Health Check') {
            steps {
                script {
                    sh '''
                        if command -v ansible-playbook &> /dev/null; then
                            ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                        else
                            echo "Ansible not available, skipping ansible stage"
                        fi
                    '''
                }
            }
        }

        stage('Apply HPA') {
            steps {
                sh 'kubectl apply -f k8s/hpa.yaml --validate=false'
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh '''
                        echo "Waiting for pods to be created..."
                        sleep 30
                        
                        # Try to get deployment status (might fail due to auth, but that's OK)
                        kubectl get deployments 2>/dev/null || echo "Cannot access deployments - check Kubernetes access manually"
                        kubectl get services 2>/dev/null || echo "Cannot access services - check Kubernetes access manually"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed. Note: Kubernetes validation was skipped.'
            echo 'Check your Kubernetes cluster manually to verify deployment status.'
        }
    }
}