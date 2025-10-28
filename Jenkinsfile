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
        
        stage('Check Available Tools') {
            steps {
                sh '''
                    echo "=== CHECKING AVAILABLE TOOLS ==="
                    which curl || echo "curl not found"
                    which wget || echo "wget not found"
                    which python3 || echo "python3 not found"
                    which python || echo "python not found"
                '''
            }
        }
        
        stage('Install Python from Binary') {
            steps {
                sh '''
                    echo "=== INSTALLING PYTHON USING CURL ==="
                    
                    # Download Python using curl
                    curl -L -o Python-3.9.18.tgz https://www.python.org/ftp/python/3.9.18/Python-3.9.18.tgz
                    
                    # Extract Python
                    tar -xzf Python-3.9.18.tgz
                    cd Python-3.9.18
                    
                    # Configure and install in user directory
                    ./configure --prefix=$HOME/python --enable-optimizations
                    make && make install
                    
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
                    
                    # Verify ansible installation
                    $HOME/python/bin/ansible --version
                '''
            }
        }
        
        stage('Run Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING ANSIBLE PLAYBOOK ==="
                    export PATH="$HOME/python/bin:$PATH"
                    cd /var/jenkins_home/workspace/jenkins-ansible-k8s-autoscale
                    ansible-playbook -i ansible/inventory ansible/playbooks/node-recovery-demo.yml --check -v
                '''
            }
        }
    }
    
    post {
        always {
            echo "🎉 JENKINS ANSIBLE AUTOMATION COMPLETED!"
        }
    }
}
