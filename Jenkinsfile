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
        
        stage('Install Standalone Python') {
            steps {
                sh '''
                    echo "=== INSTALLING STANDALONE PYTHON ==="
                    
                    # Clean any previous attempts
                    rm -rf $HOME/python
                    
                    # Download from a reliable source
                    curl -L -o python.tar.gz https://github.com/indygreg/python-build-standalone/releases/download/20230826/cpython-3.9.18+20230826-x86_64-unknown-linux-gnu-install_only.tar.gz
                    
                    # Create directory and extract
                    mkdir -p $HOME/python
                    tar -xzf python.tar.gz -C $HOME/python --strip-components=1
                    
                    echo "=== VERIFYING PYTHON ==="
                    $HOME/python/bin/python3.9 --version || $HOME/python/bin/python3 --version
                '''
            }
        }
        
        stage('Install Ansible') {
            steps {
                sh '''
                    echo "=== INSTALLING ANSIBLE ==="
                    export PATH="$HOME/python/bin:$PATH"
                    
                    # Install pip
                    curl -sS https://bootstrap.pypa.io/get-pip.py -o get-pip.py
                    python3 get-pip.py
                    
                    # Install ansible
                    pip3 install ansible
                    
                    echo "=== FINAL VERIFICATION ==="
                    python3 --version
                    pip3 --version
                    ansible --version
                    echo "✅ All dependencies installed successfully!"
                '''
            }
        }
        
        stage('Run Ansible Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING ANSIBLE PLAYBOOK ==="
                    export PATH="$HOME/python/bin:$PATH"
                    ansible-playbook -i ansible/inventory ansible/playbooks/node-recovery-demo.yml --check -v
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
