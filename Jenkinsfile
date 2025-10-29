pipeline {
    agent any

    environment {
        KUBECONFIG = '/tmp/jenkins-kube/config'
    }

    stages {
        stage('Initialize') {
            steps {
                echo '🚀 Starting Jenkins CI/CD pipeline...'
                sh 'pwd && whoami'
            }
        }

        stage('Setup Kubeconfig in Temp') {
            steps {
                echo '⚙️ Setting up Kubernetes config...'
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        mkdir -p /tmp/jenkins-kube
                        cp $KUBECONFIG_FILE /tmp/jenkins-kube/config
                        chmod 600 /tmp/jenkins-kube/config
                    '''
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                echo '⚙️ Running Ansible playbook for node recovery...'
                sh '''
                    apt-get update -y
                    apt-get install -y ansible python3 python3-pip sshpass
                    ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                '''
            }
        }
    }
}