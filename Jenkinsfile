pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Git Checkout') {
            steps {
                script {
                    // Force clean checkout
                    checkout scm: [
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        extensions: [[$class: 'CleanBeforeCheckout']],
                        userRemoteConfigs: [[url: 'https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git']]
                    ]
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }

        stage('Run Ansible') {
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