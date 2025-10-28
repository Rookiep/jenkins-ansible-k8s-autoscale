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
                        # Install kubectl
                        echo "Installing kubectl..."
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                        mkdir -p /usr/local/bin
                        mv kubectl /usr/local/bin/kubectl
                        
                        # Verify installation
                        echo "Kubectl version:"
                        /usr/local/bin/kubectl version --client
                    '''
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    sh '''
                        # Use full path to kubectl
                        /usr/local/bin/kubectl apply -f k8s/deployment.yaml
                        /usr/local/bin/kubectl apply -f k8s/service.yaml
                        
                        # Verify deployment
                        /usr/local/bin/kubectl get pods
                    '''
                }
            }
        }

        stage('Run Ansible Node Health Check') {
            steps {
                script {
                    sh '''
                        # Check if ansible is available
                        ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml || echo "Ansible not available or playbook failed"
                    '''
                }
            }
        }

        stage('Apply HPA') {
            steps {
                sh '/usr/local/bin/kubectl apply -f k8s/hpa.yaml'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
        }
    }
}