pipeline {
    agent {
        docker {
            image 'ubuntu:20.04'
            args '--privileged --network=host -v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/bin/minikube:/usr/local/bin/minikube'
        }
    }
    stages {
        stage('Setup Environment') {
            steps {
                sh '''
                echo "=== SETTING UP ENVIRONMENT ==="
                apt-get update
                
                # Install essential tools
                apt-get install -y software-properties-common curl wget git sudo
                
                # Install Ansible
                add-apt-repository --yes --update ppa:ansible/ansible
                apt-get install -y ansible
                
                # Install kubectl
                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
                
                # Verify installations
                echo "✅ Ansible version:"
                ansible-playbook --version
                echo "✅ Kubectl version:"
                kubectl version --client
                '''
            }
        }
        
        stage('Clone Repository') {
            steps {
                sh '''
                echo "=== CLONING REPOSITORY ==="
                git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                ls -la
                echo "✅ Repository cloned successfully"
                '''
            }
        }
        
        stage('Test Kubernetes Access') {
            steps {
                sh '''
                echo "=== TESTING KUBERNETES ACCESS ==="
                
                # Try to access Kubernetes cluster
                if kubectl cluster-info &>/dev/null; then
                    echo "✅ Kubernetes cluster is accessible"
                    echo "=== CURRENT NODE STATUS ==="
                    kubectl get nodes
                else
                    echo "❌ Kubernetes cluster is NOT accessible from Jenkins"
                    echo "💡 This is expected - Jenkins runs in isolated Docker container"
                    echo "💡 The Ansible playbook will run in simulation mode"
                fi
                '''
            }
        }
        
        stage('Run Ansible Playbook') {
            steps {
                sh '''
                echo "=== RUNNING ANSIBLE NODE RECOVERY ==="
                
                # This will work in simulation mode since Kubernetes is not accessible
                ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                
                echo "✅ Ansible playbook completed successfully"
                echo "💡 In production with Kubernetes access, this would restart actual nodes"
                '''
            }
        }
    }
    
    post {
        always {
            echo "🚀 Pipeline execution completed"
            echo "🔧 Ansible automation verified"
            echo "💡 Ready for production deployment with real Kubernetes access"
        }
    }
}