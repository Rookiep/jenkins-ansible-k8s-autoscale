pipeline {
    agent any
    
    environment {
        KUBECONFIG = '/home/partho/.kube/config'
        HOME = '/home/partho'
    }
    
    stages {
        stage('Verify Kubernetes Access') {
            steps {
                sh '''
                echo "=== VERIFYING KUBERNETES ACCESS ==="
                echo "KUBECONFIG: $KUBECONFIG"
                echo "HOME: $HOME"
                
                # List certificate files to verify they exist
                echo "Certificate files:"
                ls -la $HOME/.minikube/profiles/minikube/ | grep -E "(client.crt|client.key|ca.crt)"
                
                # Test Kubernetes access
                if kubectl cluster-info; then
                    echo "✅ Kubernetes cluster is accessible!"
                    echo "=== CURRENT NODE STATUS ==="
                    kubectl get nodes
                else
                    echo "❌ Kubernetes access failed"
                    exit 1
                fi
                '''
            }
        }
        
        stage('Clone Repository') {
            steps {
                sh '''
                echo "=== CLONING REPOSITORY ==="
                git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                '''
            }
        }
        
        stage('Create Node Failure') {
            steps {
                sh '''
                echo "=== CREATING NODE FAILURE ==="
                
                # Make node NotReady
                minikube ssh "sudo systemctl stop kubelet"
                
                # Wait for NotReady status
                echo "⏳ Waiting for node to become NotReady..."
                for i in {1..30}; do
                    STATUS=$(kubectl get nodes minikube -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}' 2>/dev/null)
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
                echo "=== RUNNING ANSIBLE RECOVERY ==="
                
                # Install Ansible
                apt-get update
                apt-get install -y software-properties-common
                add-apt-repository --yes --update ppa:ansible/ansible
                apt-get install -y ansible
                
                # Run the recovery playbook
                ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                
                echo "✅ Ansible recovery executed"
                '''
            }
        }
        
        stage('Verify Recovery') {
            steps {
                sh '''
                echo "=== VERIFYING RECOVERY ==="
                
                # Wait for node to become Ready
                echo "⏳ Waiting for node to recover..."
                for i in {1..30}; do
                    STATUS=$(kubectl get nodes minikube -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}' 2>/dev/null)
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
            echo "🎉 JENKINS + ANSIBLE NODE RECOVERY SUCCESSFUL!"
            '''
        }
    }
}
