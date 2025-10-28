pipeline {
    agent any

    environment {
        KUBECONFIG = '/home/jenkins/.kube/config'
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

 stage('Install Prerequisites') {
    steps {
        echo "🔧 Installing kubectl, Python & Ansible if missing..."
        sh '''
            apt-get update -y
            apt-get install -y python3 python3-pip ansible curl
            curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
            chmod +x kubectl && mv kubectl /usr/local/bin/
        '''
    }
}

        stage('Verify Kubernetes Connectivity') {
            steps {
                sh '''
                    echo "=== Checking Kubernetes Context ==="
                    if ! kubectl config current-context; then
                        echo "⚠️ No Kubernetes context found — please mount ~/.kube/config"
                    else
                        echo "✅ Kubernetes context verified"
                    fi
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
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed — please check the logs above.'
        }
    }
}