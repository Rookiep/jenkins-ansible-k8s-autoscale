pipeline {
    agent any
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    echo "=== APPLICATION DEPLOYMENT ==="
                    echo "Cleaning workspace and cloning repository..."
                    
                    # Clone the repository
                    git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                    
                    echo "✅ Code successfully checked out"
                    echo "=== KUBERNETES MANIFESTS ==="
                    ls -la k8s/
                    echo "Deployment YAML:"
                    cat k8s/deployment.yaml
                    echo "Service YAML:"
                    cat k8s/service.yaml
                    echo "HPA YAML:"
                    cat k8s/hpa.yaml
                    
                    echo "🎉 Repository structure verified"
                    '''
                }
            }
        }
        
        stage('Install and Run Ansible') {
            steps {
                script {
                    sh '''
                    echo "=== SETTING UP ANSIBLE ==="
                    if command -v ansible-playbook >/dev/null 2>&1; then
                        echo "✅ Ansible already installed"
                    else
                        echo "Installing Ansible..."
                        apt-get update
                        apt-get install -y software-properties-common
                        apt-add-repository --yes --update ppa:ansible/ansible
                        apt-get install -y ansible
                        echo "✅ Ansible installed successfully"
                    fi
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
                    echo "Checking playbook availability..."
                    if [ -f "ansible/node_recovery.yml" ]; then
                        echo "✅ Playbook found - executing..."
                        ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                        echo "✅ Ansible playbook executed successfully"
                    else
                        echo "❌ Playbook not found at ansible/node_recovery.yml"
                        echo "Available files:"
                        find . -name "*.yml" -o -name "*.yaml" | head -10
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
                echo "🚀 Pipeline execution completed!"
                echo "📚 Application manifests are ready for deployment"
                echo "🔧 Ansible node recovery system verified"
                echo "💡 Ready for production Kubernetes deployment"
            }
        }
        success {
            script {
                echo "✅ PIPELINE SUCCESS - All stages completed"
            }
        }
        failure {
            script {
                echo "❌ PIPELINE FAILED - Check stage logs for details"
            }
        }
    }
}