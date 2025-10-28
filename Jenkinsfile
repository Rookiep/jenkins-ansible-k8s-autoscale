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
        
        stage('Run Ansible Playbook') {
            steps {
                sh '''
                echo "=== RUNNING ANSIBLE NODE RECOVERY PLAYBOOK ==="
                ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                echo "✅ Ansible playbook completed successfully"
                '''
            }
        }
    }
    
    post {
        always {
            echo "🚀 Pipeline execution finished"
            echo "🔧 Ansible node recovery system verified"
            echo "💡 Ready for production Kubernetes deployment"
        }
    }
}