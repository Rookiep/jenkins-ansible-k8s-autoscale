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
                git branch: 'main', 
                url: 'https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git'
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
        
        stage('Run Correct Ansible Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING ANSIBLE PLAYBOOK ==="
                    export PATH="$HOME/python/bin:$PATH"
                    
                    # First, let's see what playbooks are available
                    echo "Available playbooks:"
                    find . -name "*.yml" -o -name "*.yaml" | grep -v ".git"
                    
                    # Try running a playbook based on actual structure
                    if [ -f "ansible/playbooks/node-recovery-demo.yml" ]; then
                        ansible-playbook -i ansible/inventory ansible/playbooks/node-recovery-demo.yml --check -v
                    elif [ -f "playbooks/node-recovery-demo.yml" ]; then
                        ansible-playbook -i inventory playbooks/node-recovery-demo.yml --check -v
                    elif [ -f "node-recovery-demo.yml" ]; then
                        ansible-playbook -i inventory node-recovery-demo.yml --check -v
                    else
                        echo "No playbook found, creating a simple test playbook"
                        cat > test-playbook.yml << 'EOF'
---
- name: Test Ansible Installation
  hosts: localhost
  connection: local
  tasks:
    - name: Display success message
      debug:
        msg: "✅ Ansible is working correctly in Jenkins!"
    - name: Show Python info
      debug:
        msg: "Python version: {{ ansible_python_version }}"
EOF
                        ansible-playbook -i localhost, test-playbook.yml
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
    }
}
