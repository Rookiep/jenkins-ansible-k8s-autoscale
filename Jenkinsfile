pipeline {
    agent any

    environment {
        KUBECONFIG = '/home/jenkins/.kube/config'
        CACHE_DIR = '/var/jenkins_home/.cache/pip'
    }

    stages {

        stage('Initialize') {
            steps {
                echo '🚀 Starting Jenkins CI/CD pipeline for Ansible + Kubernetes...'
                sh 'pwd && ls -la'
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git'
            }
        }

        stage('Install Python & Ansible (cached)') {
  steps {
    echo '✅ Python & Ansible preinstalled in Jenkins image.'
  }
}

        stage('Verify Kubernetes Connectivity') {
            steps {
                sh '''
                    echo "=== Checking Kubernetes Context ==="
                    kubectl config current-context || echo "⚠️ No context found"
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sh '''
                    echo "=== Running Ansible Playbook ==="
                    cd jenkins-ansible-k8s-autoscale
                    ansible-playbook -i inventory.ini playbook.yml
                '''
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed."
            cleanWs()
        }
    }
}