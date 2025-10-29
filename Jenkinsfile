pipeline {
    agent any

    stages {
        stage('Initialize') {
            steps {
                echo '🚀 Starting Jenkins CI/CD pipeline...'
                sh '''
                    pwd
                    whoami
                    echo "Workspace contents:"
                    ls -la
                '''
            }
        }

        stage('Setup Kubeconfig in Temp') {
            steps {
                echo '⚙️ Setting up Kubernetes config...'
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                        sh '''
                            echo "📁 Preparing kubeconfig directory..."
                            mkdir -p /tmp/jenkins-kube
                            cp "$KUBECONFIG_FILE" /tmp/jenkins-kube/config
                            chmod 600 /tmp/jenkins-kube/config
                            echo "✅ Kubeconfig successfully placed at /tmp/jenkins-kube/config"
                            ls -la /tmp/jenkins-kube
                            
                            export KUBECONFIG="/tmp/jenkins-kube/config"
                            echo "KUBECONFIG set to: $KUBECONFIG"
                        '''
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '🔧 Installing dependencies...'
                sh '''
                    apt-get update -y
                    apt-get install -y curl ca-certificates python3 python3-pip ansible
                    
                    # Install kubectl
                    K8S_VERSION=v1.30.0
                    curl -LO "https://dl.k8s.io/release/$K8S_VERSION/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/
                    echo "✅ kubectl version:"
                    kubectl version --client
                '''
            }
        }

        stage('Test Kubeconfig Access') {
            steps {
                echo '🔍 Testing kubeconfig access...'
                sh '''
                    export KUBECONFIG="/tmp/jenkins-kube/config"
                    echo "Testing kubectl access with the configured kubeconfig..."
                    
                    if kubectl cluster-info 2>/dev/null; then
                        echo "✅ Successfully connected to Kubernetes cluster"
                        kubectl get nodes
                    else
                        echo "❌ Cannot connect to Kubernetes cluster"
                        echo "Kubeconfig contents (first few lines):"
                        head -20 /tmp/jenkins-kube/config
                        echo "=== Debug Information ==="
                        kubectl config view --kubeconfig=/tmp/jenkins-kube/config
                    fi
                '''
            }
        }

        stage('Fix and Run Ansible Playbook') {
            steps {
                echo '⚙️ Fixing and running Ansible playbook...'
                sh '''
                    export KUBECONFIG=/tmp/jenkins-kube/config
                    echo "🧩 Switching context to docker-desktop..."
                    kubectl config use-context docker-desktop --kubeconfig=$KUBECONFIG || echo "⚠️ docker-desktop context not found."

                    echo "🔍 Testing cluster access..."
                    kubectl cluster-info || echo "❌ Still cannot connect — check if Docker Desktop Kubernetes is enabled."

                    echo "📄 Creating fixed Ansible playbook..."
                    cat <<'EOF' > ansible/node_recovery_fixed.yml
---
- name: Kubernetes Node Auto-Recovery (Docker Desktop)
  hosts: localhost
  connection: local
  gather_facts: no

  vars:
    kubeconfig_path: "/tmp/jenkins-kube/config"

  tasks:
    - name: Check Kubernetes cluster status
      shell: kubectl cluster-info --kubeconfig {{ kubeconfig_path }}
      register: cluster_status
      ignore_errors: yes
      environment:
        KUBECONFIG: "{{ kubeconfig_path }}"

    - name: Abort if Kubernetes cluster not accessible
      fail:
        msg: |
          ❌ Cluster unreachable.
          💡 Ensure Docker Desktop Kubernetes is enabled.
      when: cluster_status.rc != 0

    - name: Display successful cluster connection
      debug:
        msg: "✅ Kubernetes cluster reachable using {{ kubeconfig_path }}."
EOF

                    echo "🚀 Running fixed Ansible playbook..."
                    ansible-playbook -i ansible/inventory.ini ansible/node_recovery_fixed.yml -v
                '''
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up...'
            sh '''
                rm -rf /tmp/jenkins-kube 2>/dev/null || true
                echo "Cleanup completed"
            '''
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}