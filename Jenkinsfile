pipeline {
    agent any
    
    stages {
        stage('Clone Repository') {
            steps {
                sh '''
                echo "=== CLONING REPOSITORY ==="
                git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                echo "✅ Repository cloned"
                '''
            }
        }
        
        stage('Verify Environment') {
            steps {
                sh '''
                echo "=== VERIFYING ENVIRONMENT ==="
                echo "Kubernetes cluster info:"
                kubectl cluster-info
                echo "Node status:"
                kubectl get nodes
                echo "Minikube status:"
                minikube status
                echo "✅ Environment verified"
                '''
            }
        }
        
        stage('Create Test Failure') {
            steps {
                sh '''
                echo "=== CREATING TEST NODE FAILURE ==="
                
                # Make node NotReady for testing
                minikube ssh "sudo systemctl stop kubelet"
                
                # Wait for NotReady status
                echo "⏳ Waiting for node to become NotReady..."
                for i in {1..30}; do
                    STATUS=$(kubectl get nodes minikube -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}' 2>/dev/null || echo "Unknown")
                    if [ "$STATUS" = "False" ]; then
                        echo "🔴 SUCCESS: Node is now NotReady!"
                        kubectl get nodes
                        break
                    fi
                    echo "Waiting... ($i/30) - Status: $STATUS"
                    sleep 2
                done
                '''
            }
        }
        
        stage('Run Ansible Recovery') {
            steps {
                sh '''
                echo "=== RUNNING ANSIBLE NODE RECOVERY ==="
                
                # Install Ansible if not present
                if ! command -v ansible-playbook &> /dev/null; then
                    echo "Installing Ansible..."
                    apt-get update
                    apt-get install -y software-properties-common
                    add-apt-repository --yes --update ppa:ansible/ansible
                    apt-get install -y ansible
                fi
                
                # Run the recovery playbook
                ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                
                echo "✅ Ansible recovery executed"
                '''
            }
        }
        
        stage('Verify Recovery') {
            steps {
                sh '''
                echo "=== VERIFYING NODE RECOVERY ==="
                
                # Wait for node to become Ready
                echo "⏳ Waiting for node to recover..."
                for i in {1..30}; do
                    STATUS=$(kubectl get nodes minikube -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}' 2>/dev/null || echo "Unknown")
                    if [ "$STATUS" = "True" ]; then
                        echo "✅ SUCCESS: Node is now Ready!"
                        kubectl get nodes
                        break
                    fi
                    echo "Waiting for recovery... ($i/30) - Status: $STATUS"
                    sleep 2
                done
                '''
            }
        }
    }
    
    post {
        always {
            sh '''
            echo "=== FINAL STATUS ==="
            kubectl get nodes
            echo "🎉 PIPELINE EXECUTION COMPLETED"
            '''
        }
        success {
            echo "✅ NODE RECOVERY SUCCESSFUL - Jenkins + Ansible working perfectly!"
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
        }
    }
}
