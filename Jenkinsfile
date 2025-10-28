pipeline {
    agent {
        label 'python'  // Use an agent with Python pre-installed
    }
    
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
        
        stage('Verify Python') {
            steps {
                sh '''
                    echo "=== CHECKING PYTHON ==="
                    python3 --version
                    pip3 --version
                '''
            }
        }
        
        stage('Install Ansible') {
            steps {
                sh '''
                    echo "=== INSTALLING ANSIBLE ==="
                    pip3 install ansible
                '''
            }
        }
        
        stage('Run Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING ANSIBLE PLAYBOOK ==="
                    ansible-playbook -i ansible/inventory ansible/playbooks/node-recovery-demo.yml --check
                '''
            }
        }
    }
}
