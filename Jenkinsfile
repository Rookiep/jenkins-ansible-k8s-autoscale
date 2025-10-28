pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git'
            }
        }

        stage('Explore Repository Structure') {
            steps {
                sh '''
                    echo "=== EXPLORING REPOSITORY STRUCTURE ==="
                    pwd
                    ls -la
                    echo "=== ANSIBLE DIRECTORY CONTENTS ==="
                    ls -la ansible/ || echo "No ansible directory"
                    echo "=== PLAYBOOKS DIRECTORY CONTENTS ==="
                    find . -name "*.yml" -o -name "*.yaml" | head -20
                    echo "=== INVENTORY FILES ==="
                    find . -name "inventory*" | head -10
                '''
            }
        }

        stage('Setup Kubeconfig') {
            steps {
                sh '''
                    echo "=== SETTING UP KUBECONFIG ==="
                    mkdir -p $HOME/.kube

                    # Try to detect kubeconfig from multiple paths
                    if [ -f "$HOME/.kube/config" ]; then
                        echo "✅ Using existing kubeconfig at $HOME/.kube/config"
                    elif [ -f "/root/.kube/config" ]; then
                        echo "📁 Copying from /root/.kube/config"
                        cp /root/.kube/config $HOME/.kube/config
                    elif [ -f "/mnt/c/Users/ACER/.kube/config" ]; then
                        echo "📁 Copying from Windows path /mnt/c/Users/ACER/.kube/config"
                        cp /mnt/c/Users/ACER/.kube/config $HOME/.kube/config
                    else
                        echo "⚠️ No kubeconfig found. Run 'minikube start' first."
                    fi

                    echo "=== VERIFYING kubeconfig ==="
                    cat $HOME/.kube/config || echo "No kubeconfig file detected"
                '''
            }
        }

        stage('Install Standalone Python') {
            steps {
                sh '''
                    echo "=== INSTALLING STANDALONE PYTHON ==="
                    rm -rf $HOME/python
                    curl -L -o python.tar.gz https://github.com/indygreg/python-build-standalone/releases/download/20230826/cpython-3.9.18+20230826-x86_64-unknown-linux-gnu-install_only.tar.gz
                    mkdir -p $HOME/python
                    tar -xzf python.tar.gz -C $HOME/python --strip-components=1
                    $HOME/python/bin/python3.9 --version
                '''
            }
        }

        stage('Install Ansible') {
            steps {
                sh '''
                    echo "=== INSTALLING ANSIBLE ==="
                    export PATH="$HOME/python/bin:$PATH"
                    curl -sS https://bootstrap.pypa.io/get-pip.py -o get-pip.py
                    python3 get-pip.py
                    pip3 install ansible
                    echo "=== VERIFICATION ==="
                    python3 --version
                    pip3 --version
                    ansible --version
                '''
            }
        }

        stage('Run Node Recovery Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING NODE RECOVERY PLAYBOOK ==="
                    export PATH="$HOME/python/bin:$PATH"
                    export KUBECONFIG="$HOME/.kube/config"

                    if [ -f "ansible/node_recovery.yml" ]; then
                        echo "🚀 Running Ansible playbook: ansible/node_recovery.yml"
                        ansible-playbook -i localhost, ansible/node_recovery.yml
                    else
                        echo "❌ node_recovery.yml not found. Please verify repository structure."
                        exit 1
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "🎉 JENKINS ANSIBLE AUTOMATION COMPLETED!"
        }
        success {
            echo "✅ SUCCESS - Pipeline executed successfully!"
        }
        failure {
            echo "❌ FAILURE - Pipeline encountered an error!"
        }
    }
}
