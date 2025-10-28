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

        stage('Verify Checkout') {
            steps {
                sh 'git status && ls -la'
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
                            sudo mv kubectl /usr/local/bin/
                        else
                            echo "kubectl is already installed"
                        fi
                        
                        # Verify installation
                        kubectl version --client
                    '''
                }
            }
        }

        stage('Configure K8s Access') {
            steps {
                script {
                    // Method 1: If you have kubeconfig file in your repository
                    sh '''
                        # Copy kubeconfig to appropriate location
                        if [ -f "kubeconfig" ]; then
                            mkdir -p ~/.kube
                            cp kubeconfig ~/.kube/config
                            chmod 600 ~/.kube/config
                        fi
                    '''
                    
                    // Method 2: Use Jenkins credentials for kubeconfig
                    withCredentials([file(credentialsId: 'k8s-kubeconfig', variable: 'KUBECONFIG')]) {
                        sh '''
                            mkdir -p ~/.kube
                            cp $KUBECONFIG ~/.kube/config
                            chmod 600 ~/.kube/config
                        '''
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    sh '''
                        # Test kubectl connection
                        kubectl cluster-info
                        kubectl get nodes
                        
                        # Deploy applications
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }

        stage('Run Ansible Node Health Check') {
            steps {
                script {
                    sh '''
                        # Install ansible if not present (optional)
                        if ! command -v ansible &> /dev/null; then
                            echo "Ansible not found, installing..."
                            apt-get update && apt-get install -y ansible
                        fi
                        
                        ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                    '''
                }
            }
        }

        stage('Apply HPA') {
            steps {
                sh 'kubectl apply -f k8s/hpa.yaml'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
        }
    }
}