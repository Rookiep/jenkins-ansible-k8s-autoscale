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
                sh '''
                    echo "=== Installing Standalone Python if missing ==="
                    if [ ! -d "/var/jenkins_home/python" ]; then
                        curl -L -o python.tar.gz https://github.com/indygreg/python-build-standalone/releases/download/20230826/cpython-3.9.18+20230826-x86_64-unknown-linux-gnu-install_only.tar.gz
                        mkdir -p /var/jenkins_home/python
                        tar -xzf python.tar.gz -C /var/jenkins_home/python --strip-components=1
                    fi
                    export PATH=/var/jenkins_home/python/bin:$PATH
                    echo "Python version:" && python3 --version

                    echo "=== Installing Ansible with caching ==="
                    mkdir -p $CACHE_DIR
                    python3 -m pip install --upgrade pip wheel
                    pip3 install --cache-dir=$CACHE_DIR --prefer-binary ansible==8.7.0
                    ansible --version
                '''
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
                    export PATH=/var/jenkins_home/python/bin:$PATH
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