pipeline {
    agent any
    
    stages {
        stage('Clean and Setup') {
            steps {
                cleanWs()
                sh '''
                git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                '''
            }
        }
        
        stage('Verify Files') {
            steps {
                sh '''
                echo "=== VERIFYING FILES ==="
                ls -la
                echo "K8S files:"
                ls -la k8s/
                echo "Ansible files:"
                ls -la ansible/
                '''
            }
        }
        
        stage('Run Ansible') {
            steps {
                sh '''
                echo "=== RUNNING ANSIBLE ==="
                ansible-playbook --version
                ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                '''
            }
        }
    }
    
    post {
        always {
            echo "✅ Pipeline completed"
        }
    }
}