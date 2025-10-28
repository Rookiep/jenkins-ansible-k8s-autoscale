pipeline {
    agent any  // Run on Jenkins host directly
    
    stages {
        stage('Restart Node') {
            steps {
                sh '''
                echo "=== RESTARTING KUBERNETES NODE ==="
                
                # Make node NotReady
                minikube ssh "sudo systemctl stop kubelet"
                
                # Wait for NotReady status
                echo "⏳ Waiting for node to become NotReady..."
                until kubectl get nodes minikube -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}' | grep -q "False"; do
                    sleep 5
                done
                
                echo "🔴 Node is now NotReady - running recovery..."
                
                # Run Ansible recovery
                ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                
                echo "✅ Node recovery completed!"
                kubectl get nodes
                '''
            }
        }
    }
}