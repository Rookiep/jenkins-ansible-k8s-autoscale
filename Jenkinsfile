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
            echo "🧩 Installing dependencies..."
            apt-get update -y
            apt-get install -y curl python3 python3-pip ansible

            echo "📦 Installing kubectl..."
            curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
            chmod +x kubectl
            mv kubectl /usr/local/bin/

            echo "✅ kubectl version:"
            kubectl version --client

            echo "🚀 Running Ansible Playbook..."
            ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
        '''
    }
}
    }