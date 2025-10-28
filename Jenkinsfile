pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git'
            }
        }

        stage('Install Python & Ansible') {
            steps {
                sh '''
                    echo "=== Installing Standalone Python ==="
                    rm -rf $HOME/python
                    curl -L -o python.tar.gz https://github.com/indygreg/python-build-standalone/releases/download/20230826/cpython-3.9.18+20230826-x86_64-unknown-linux-gnu-install_only.tar.gz
                    mkdir -p $HOME/python
                    tar -xzf python.tar.gz -C $HOME/python --strip-components=1
                    export PATH="$HOME/python/bin:$PATH"

                    echo "=== Installing Ansible ==="
                    curl -sS https://bootstrap.pypa.io/get-pip.py -o get-pip.py
                    python3 get-pip.py
                    pip3 install ansible

                    echo "=== Verification ==="
                    ansible --version
                '''
            }
        }

        stage('Run Node Recovery Playbook') {
            steps {
                sh '''
                    echo "=== Executing Node Recovery ==="
                    export PATH="$HOME/python/bin:$PATH"

                    if [ -f "ansible/node_recovery.yml" ]; then
                        echo "🚀 Running ansible/node_recovery.yml"
                        ansible-playbook -i localhost, ansible/node_recovery.yml -v
                    else
                        echo "❌ node_recovery.yml not found!"
                        exit 1
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "🎉 Jenkins Ansible automation completed."
        }
        success {
            echo "✅ Kubernetes node recovery completed successfully!"
        }
        failure {
            echo "❌ Jenkins build failed during node recovery."
        }
    }
}