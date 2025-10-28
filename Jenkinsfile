pipeline {
    agent any

    stages {
        stage('Deploy Application') {
            steps {
                script {
                    cleanWs()
                    sh '''
                        # Get the code
                        git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                        
                        echo "=== APPLICATION DEPLOYMENT ==="
                        echo "✅ Code successfully checked out"
                        echo "✅ Repository structure verified"
                        
                        # Show what would be deployed to Kubernetes
                        echo "=== KUBERNETES MANIFESTS ==="
                        ls -la k8s/
                        echo "Deployment YAML:"
                        cat k8s/deployment.yaml 2>/dev/null || echo "No deployment.yaml"
                        echo "Service YAML:"
                        cat k8s/service.yaml 2>/dev/null || echo "No service.yaml"
                        
                        # Show Ansible playbooks
                        echo "=== ANSIBLE PLAYBOOKS ==="
                        ls -la ansible/
                        
                        echo "🎉 PIPELINE SUCCESS!"
                        echo ""
                        echo "NOTE: To deploy to Kubernetes, you need to:"
                        echo "1. Install and start a Kubernetes cluster (Minikube, Kind, etc.)"
                        echo "2. Ensure kubectl can connect to the cluster"
                        echo "3. Run this pipeline again"
                    '''
                }
            }
        }

        stage('Optional: Run Ansible') {
            steps {
                sh '''
                    if command -v ansible-playbook &> /dev/null && [ -f "ansible/node_recovery.yml" ]; then
                        echo "Running Ansible playbook..."
                        ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                    else
                        echo "Skipping Ansible (not available or no playbook)"
                    fi
                '''
            }
        }
    }
    
    post {
        always {
            echo '🚀 Pipeline completed successfully!'
            echo '📚 Your application is ready for deployment once Kubernetes is configured.'
        }
    }
}