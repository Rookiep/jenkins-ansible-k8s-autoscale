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
        
        stage('Download Pre-compiled Python') {
            steps {
                sh '''
                    echo "=== DOWNLOADING PRE-COMPILED PYTHON ==="
                    
                    # Download standalone Python binary
                    curl -L -o python-standalone.tar.gz https://github.com/indygreg/python-build-standalone/releases/download/20230826/cpython-3.9.18+20230826-x86_64-unknown-linux-gnu-install_only.tar.gz
                    
                    # Extract to home directory
                    mkdir -p $HOME/python
                    tar -xzf python-standalone.tar.gz -C $HOME/python --strip-components=1
                    
                    # Verify installation
                    $HOME/python/bin/python3 --version
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
                    $HOME/python/bin/python3 get-pip.py
                    
                    # Install ansible
                    $HOME/python/bin/pip3 install ansible
                    
                    echo "=== VERIFICATION ==="
                    $HOME/python/bin/python3 --version
                    $HOME/python/bin/pip3 --version
                    $HOME/python/bin/ansible --version
                '''
            }
        }
        
        stage('Run Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING ANSIBLE PLAYBOOK ==="
                    export PATH="$HOME/python/bin:$PATH"
                    ansible-playbook -i ansible/inventory ansible/playbooks/node-recovery-demo.yml --check
                '''
            }
        }
    }
}
