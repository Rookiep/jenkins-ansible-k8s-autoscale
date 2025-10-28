pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
                sh 'pwd && ls -la'
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    sh '''
                        git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                        ls -la
                    '''
                }
            }
        }

        stage('Install kubectl') {
            steps {
                script {
                    sh '''
                        echo "Installing kubectl..."
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                        mkdir -p /usr/local/bin
                        mv kubectl /usr/local/bin/kubectl
                        echo "Kubectl version:"
                        /usr/local/bin/kubectl version --client
                    '''
                }
            }
        }

        stage('Configure Kubernetes Access') {
            steps {
                script {
                    sh '''
                        # Create .kube directory
                        mkdir -p ~/.kube
                        
                        # Check if we're in a Kubernetes cluster (like in a pod)
                        if [ -f "/var/run/secrets/kubernetes.io/serviceaccount/token" ]; then
                            echo "Running inside Kubernetes cluster, configuring in-cluster access..."
                            # In-cluster configuration
                            kubectl config set-cluster in-cluster --server=https://kubernetes.default.svc --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
                            kubectl config set-credentials jenkins --token=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
                            kubectl config set-context jenkins-context --cluster=in-cluster --user=jenkins
                            kubectl config use-context jenkins-context
                        else
                            echo "Configuring local Kubernetes access..."
                            
                            # For Minikube
                            if command -v minikube &> /dev/null; then
                                echo "Using minikube configuration"
                                minikube update-context
                            fi
                            
                            # For Docker Desktop K8s
                            if [ -f "$HOME/.kube/config" ]; then
                                echo "Using existing kubeconfig"
                            else
                                echo "No kubeconfig found. Please configure Kubernetes access."
                            fi
                        fi
                        
                        # Test connection
                        echo "Testing Kubernetes connection..."
                        kubectl cluster-info || echo "Cannot connect to cluster - authentication required"
                        kubectl get nodes || echo "Cannot list nodes - check permissions"
                    '''
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    sh '''
                        echo "Current kubeconfig context:"
                        kubectl config current-context
                        
                        echo "Available contexts:"
                        kubectl config get-contexts
                        
                        # Deploy with validation disabled initially
                        echo "Deploying to Kubernetes..."
                        kubectl apply -f k8s/deployment.yaml --validate=false
                        kubectl apply -f k8s/service.yaml --validate=false
                        
                        # Verify deployment
                        sleep 10
                        kubectl get deployments,services,pods
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
                            echo "Ansible not installed, skipping ansible stage"
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
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            sh 'kubectl get all 2>/dev/null || echo "Kubernetes not accessible"'
        }
    }
}