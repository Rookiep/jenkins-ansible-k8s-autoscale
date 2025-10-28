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
                    
                    echo "=== INSTALLING PYTHON3 AND PIP ==="
                    apt-get update
                    apt-get install -y python3 python3-pip
                    
                    echo "=== INSTALLING ANSIBLE ==="
                    pip3 install ansible
                '''
            }
        }
        
        stage('Verify Installation') {
            steps {
                sh '''
                    echo "=== VERIFYING INSTALLATIONS ==="
                    python3 --version
                    pip3 --version
                    ansible --version
                '''
            }
        }
        
        stage('Run Ansible Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING ANSIBLE PLAYBOOK ==="
                    cd /var/jenkins_home/workspace/jenkins-ansible-k8s-autoscale
                    ansible-playbook -i inventory playbooks/node-recovery-demo.yml --check
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
