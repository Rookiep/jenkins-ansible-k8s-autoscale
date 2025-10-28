pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/jenkins_home/.kube/config'
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
                echo '📦 Cloning repository...'
                git branch: 'main', url: 'https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git'
            }
        }

      stage('Install Prerequisites') {
    steps {
        echo '🔧 Installing kubectl, Python & Ansible if missing...'
        sh '''
            apt-get update -y
            apt-get install -y curl python3 python3-pip ansible
            curl -LO "https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl"
            chmod +x kubectl
            mv kubectl /usr/local/bin/
            kubectl version --client
        '''
    }
}

        stage('Verify Kubernetes Connectivity') {
            steps {
                echo '🔍 Verifying Kubernetes connection...'
                sh '''
                    echo "Current context:"
                    kubectl config current-context
                    echo "Available nodes:"
                    kubectl get nodes -o wide
                '''
            }
        }

        stage('Restart Node (Simulated)') {
            steps {
                echo '🔁 Simulating Kubernetes node restart...'
                sh '''
                    NODE=$(kubectl get nodes -o name | head -n1 | cut -d'/' -f2)
                    echo "Draining node: $NODE"
                    kubectl drain $NODE --ignore-daemonsets --delete-emptydir-data || true
                    sleep 5
                    echo "Uncordoning node: $NODE"
                    kubectl uncordon $NODE || true
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                echo '⚙️ Running Ansible playbook...'
                sh '''
                    ansible-playbook -i inventory.ini playbook.yml || true
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed — please check the logs above.'
        }
    }
}