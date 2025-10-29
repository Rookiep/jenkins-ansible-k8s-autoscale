pipeline {
    agent any

    environment {
        // ✅ Ensure KUBECONFIG is consistent across all stages
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
                        echo "📁 Preparing kubeconfig directory..."
                        mkdir -p /tmp/jenkins-kube
                        cp $KUBECONFIG_FILE /tmp/jenkins-kube/config
                        chmod 600 /tmp/jenkins-kube/config

                        echo "✅ Kubeconfig successfully placed at /tmp/jenkins-kube/config"
                        ls -la /tmp/jenkins-kube
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
                    apt-get install -y curl ca-certificates python3 python3-pip ansible

                    echo "📦 Installing kubectl (explicit version)..."
                    K8S_VERSION="v1.30.0"
                    curl -LO "https://dl.k8s.io/release/${K8S_VERSION}/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/

                    echo "✅ kubectl version:"
                    kubectl version --client

                    echo "🔍 Verifying kubeconfig before running playbook..."
                    if [ ! -f /tmp/jenkins-kube/config ]; then
                        echo "❌ Kubeconfig missing at /tmp/jenkins-kube/config"
                        exit 1
                    fi

                    echo "🚀 Running Ansible Playbook..."
                    export KUBECONFIG=/tmp/jenkins-kube/config
                    ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                '''
            }
        }
    }
}