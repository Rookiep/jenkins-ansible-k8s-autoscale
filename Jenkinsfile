pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // This will use the SCM configuration from your Jenkins job
                checkout scm
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }

        stage('Run Ansible Node Health Check') {
            steps {
                sh 'ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml'
            }
        }

        stage('Apply HPA') {
            steps {
                sh 'kubectl apply -f k8s/hpa.yaml'
            }
        }
    }
}