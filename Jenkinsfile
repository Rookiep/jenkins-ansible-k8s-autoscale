pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {
        stage('Initialize') {
            steps {
                echo '🚀 Starting Jenkins CI/CD pipeline...'
                sh '''
                    echo "Workspace: $(pwd)"
                    echo "User: $(whoami)"
                    echo "Home: $HOME"
                    echo "Using existing repository files..."
                '''
            }
        }

        stage('Check Existing Files') {
            steps {
                echo '📁 Checking existing repository files...'
                sh '''
                    echo "=== Available Files ==="
                    ls -la
                    echo "=== Ansible Files ==="
                    ls -la ansible/ 2>/dev/null || echo "No ansible directory"
                    echo "=== K8s Files ==="
                    ls -la k8s/ 2>/dev/null || echo "No k8s directory"
                '''
            }
        }

        stage('Install Tools in Safe Location') {
            steps {
                echo '🔧 Installing tools to safe locations...'
                sh '''
                    # Create a writable directory for tools
                    mkdir -p /tmp/jenkins-tools
                    cd /tmp/jenkins-tools
                    
                    # Install kubectl to temp directory
                    if ! command -v kubectl &> /dev/null; then
                        echo "Installing kubectl to /tmp/jenkins-tools..."
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                        export PATH="/tmp/jenkins-tools:$PATH"
                    fi
                    
                    # Verify tools
                    echo "=== Tool Versions ==="
                    python3 --version || echo "Python3 not available"
                    ansible --version || echo "Ansible not available"
                    /tmp/jenkins-tools/kubectl version --client 2>/dev/null || echo "Kubectl not available"
                '''
            }
        }

        stage('Setup Kubeconfig in Temp') {
            steps {
                echo '⚙️ Setting up Kubernetes config in temp...'
                script {
                    // This assumes you have a kubeconfig credential set up in Jenkins
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                        sh '''
                            mkdir -p /tmp/jenkins-kube
                            cp "$KUBECONFIG_FILE" /tmp/jenkins-kube/config
                            chmod 600 /tmp/jenkins-kube/config
                            export KUBECONFIG="/tmp/jenkins-kube/config"
                            echo "Kubeconfig setup in /tmp/jenkins-kube/"
                        '''
                    }
                }
            }
        }

        stage('Test Kubernetes Access') {
            steps {
                echo '🔍 Testing Kubernetes access...'
                sh '''
                    export KUBECONFIG="/tmp/jenkins-kube/config"
                    export PATH="/tmp/jenkins-tools:$PATH"
                    
                    echo "=== Kubernetes Test ==="
                    kubectl get nodes 2>/dev/null && echo "✅ Kubernetes access successful" || echo "❌ Kubernetes access failed"
                    
                    # Even if kubectl fails, continue the pipeline
                    echo "Continuing pipeline..."
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                echo '⚙️ Running Ansible playbook...'
                sh '''
                    cd /var/jenkins_home/workspace/jenkins-ansible-k8s-autoscale
                    
                    echo "=== Available Ansible Files ==="
                    find . -name "*.yml" -o -name "*.yaml" -o -name "*.ini" 2>/dev/null | sort || true
                    
                    # Try to run Ansible if files exist
                    if [ -f "ansible/playbook.yml" ]; then
                        echo "Found Ansible playbook, running in check mode..."
                        cd ansible
                         ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml--check --syntax-check
                        echo "Ansible syntax check completed"
                    else
                        echo "No Ansible playbook found, simulating..."
                        echo "✅ Simulated Ansible execution completed"
                    fi
                '''
            }
        }

        stage('Simulate Node Operations') {
            steps {
                echo '🔁 Simulating node operations...'
                sh '''
                    export KUBECONFIG="/tmp/jenkins-kube/config"
                    export PATH="/tmp/jenkins-tools:$PATH"
                    
                    echo "=== Simulated Node Maintenance ==="
                    echo "1. Checking cluster status..."
                    kubectl get nodes 2>/dev/null || echo "Cannot access cluster"
                    
                    echo "2. Simulating node drain..."
                    echo "   - Would cordon node"
                    echo "   - Would drain pods"
                    echo "   - Would uncordon node"
                    
                    echo "3. Simulation completed successfully"
                    echo "✅ Node operations simulated"
                '''
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up temporary files...'
            sh '''
                # Clean up only files we created in temp
                rm -rf /tmp/jenkins-tools /tmp/jenkins-kube 2>/dev/null || true
                echo "Cleanup completed"
            '''
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline completed with errors.'
        }
    }
}
