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
        
        stage('Install Python from Binary') {
            steps {
                sh '''
                    echo "=== INSTALLING PYTHON FROM BINARY ==="
                    
                    # Download and extract Python binary
                    wget https://www.python.org/ftp/python/3.9.18/Python-3.9.18.tgz
                    tar -xzf Python-3.9.18.tgz
                    cd Python-3.9.18
                    
                    # Configure and install in user directory
                    ./configure --prefix=$HOME/python --enable-optimizations
                    make && make install
                    
                    # Add to PATH
                    export PATH="$HOME/python/bin:$PATH"
                    echo "export PATH=\\"$HOME/python/bin:\\$PATH\\"" >> ~/.bashrc
                    
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
                    
                    # Install pip and ansible
                    curl -sS https://bootstrap.pypa.io/get-pip.py -o get-pip.py
                    $HOME/python/bin/python3 get-pip.py
                    $HOME/python/bin/pip3 install ansible
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
