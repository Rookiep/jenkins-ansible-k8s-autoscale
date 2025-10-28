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
        
        stage('Install Python & Dependencies') {
            steps {
                sh '''
                    echo "=== CHECKING CURRENT ENVIRONMENT ==="
                    whoami
                    pwd
                    ls -la
                    
                    echo "=== CHECKING EXISTING PYTHON INSTALLATION ==="
                    python3 --version || echo "Python3 not available"
                    pip3 --version || echo "Pip3 not available"
                '''
            }
        }
        
        stage('Install Ansible with User Pip') {
            steps {
                sh '''
                    echo "=== INSTALLING ANSIBLE USING GET-PIP ==="
                    # Download and install pip for user
                    curl -sS https://bootstrap.pypa.io/get-pip.py -o get-pip.py
                    python3 get-pip.py --user
                    
                    # Install ansible for user
                    python3 -m pip install --user ansible
                '''
            }
        }
        
        stage('Verify Installation') {
            steps {
                sh '''
                    echo "=== VERIFYING INSTALLATIONS ==="
                    python3 --version
                    python3 -m pip --version
                    python3 -m ansible --version
                    
                    echo "=== ADDING PIP TO PATH ==="
                    export PATH="$HOME/.local/bin:$PATH"
                    ansible --version || echo "Ansible not in PATH"
                '''
            }
        }
        
        stage('Run Ansible Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING ANSIBLE PLAYBOOK ==="
                    export PATH="$HOME/.local/bin:$PATH"
                    ansible-playbook -i ansible/inventory ansible/playbooks/node-recovery-demo.yml --check
                '''
            }
        }
    }
    
    post {
        always {
            echo "🎉 JENKINS ANSIBLE AUTOMATION COMPLETED!"
        }
        success {
            echo "✅ SUCCESS - Pipeline executed successfully"
        }
        failure {
            echo "❌ PIPELINE FAILED - Check specific stage for errors"
        }
    }
}
