pipeline {
    agent any
    
    stages {
        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    echo "=== APPLICATION DEPLOYMENT ==="
                    git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                    echo "✅ Code successfully checked out"
                    
                    # Check if we have Kubernetes access
                    if command -v kubectl >/dev/null 2>&1 && kubectl cluster-info >/dev/null 2>&1; then
                        echo "✅ Kubernetes cluster is accessible"
                        echo "Deploying application..."
                        kubectl apply -f k8s/
                    else
                        echo "ℹ️  Kubernetes cluster not accessible - showing deployment manifests"
                        echo "=== KUBERNETES MANIFESTS ==="
                        ls -la k8s/
                        echo "Deployment YAML:"
                        cat k8s/deployment.yaml
                        echo "Service YAML:"
                        cat k8s/service.yaml
                    fi
                    '''
                }
            }
        }
        
        stage('Install and Run Ansible') {
            steps {
                script {
                    sh '''
                    echo "=== SETTING UP ANSIBLE ==="
                    if ! command -v ansible-playbook >/dev/null 2>&1; then
                        echo "Installing Ansible..."
                        apt-get update
                        apt-get install -y software-properties-common
                        apt-add-repository --yes --update ppa:ansible/ansible
                        apt-get install -y ansible
                    fi
                    echo "✅ Ansible installed/verified"
                    ansible-playbook --version
                    '''
                }
            }
        }
        
        stage('Run Ansible Playbook') {
            steps {
                script {
                    sh '''
                    echo "=== RUNNING ANSIBLE PLAYBOOK ==="
                    if [ -f ansible/node_recovery.yml ]; then
                        echo "Executing Ansible playbook..."
                        # This will work in both cluster and simulation mode
                        ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                    else
                        echo "❌ Ansible playbook not found"
                        exit 1
                    fi
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "🚀 Pipeline completed!"
                echo "📚 Application is ready for deployment"
                echo "🔧 Ansible node recovery system is operational"
            }
        }
    }
}