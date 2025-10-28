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

        stage('Explore Repository Structure') {
            steps {
                sh '''
                    echo "=== EXPLORING REPOSITORY STRUCTURE ==="
                    pwd
                    ls -la
                    echo "=== ANSIBLE DIRECTORY CONTENTS ==="
                    ls -la ansible/ || echo "No ansible directory"
                    echo "=== PLAYBOOKS DIRECTORY CONTENTS ==="
                    find . -name "*.yml" -o -name "*.yaml" | head -20
                    echo "=== INVENTORY FILES ==="
                    find . -name "inventory*" | head -10
                '''
            }
        }

        stage('Install Standalone Python') {
            steps {
                sh '''
                    echo "=== INSTALLING STANDALONE PYTHON ==="
                    rm -rf $HOME/python
                    curl -L -o python.tar.gz https://github.com/indygreg/python-build-standalone/releases/download/20230826/cpython-3.9.18+20230826-x86_64-unknown-linux-gnu-install_only.tar.gz
                    mkdir -p $HOME/python
                    tar -xzf python.tar.gz -C $HOME/python --strip-components=1
                    $HOME/python/bin/python3.9 --version
                '''
            }
        }

        stage('Install Ansible') {
            steps {
                sh '''
                    echo "=== INSTALLING ANSIBLE ==="
                    export PATH="$HOME/python/bin:$PATH"
                    curl -sS https://bootstrap.pypa.io/get-pip.py -o get-pip.py
                    python3 get-pip.py
                    pip3 install ansible
                    
                    echo "=== VERIFICATION ==="
                    python3 --version
                    pip3 --version
                    ansible --version
                '''
            }
        }

        stage('Run Node Recovery Playbook') {
            steps {
                sh '''
                    echo "=== RUNNING NODE RECOVERY PLAYBOOK ==="
                    export PATH="$HOME/python/bin:$PATH"

                    if [ -f "ansible/node_recovery.yml" ]; then
                        echo "🚀 Running Ansible playbook: ansible/node_recovery.yml"
                        ansible-playbook -i localhost, ansible/node_recovery.yml
                    else
                        echo "❌ node_recovery.yml not found. Please verify repository structure."
                        exit 1
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "🎉 JENKINS ANSIBLE AUTOMATION COMPLETED!"
        }
        success {
            echo "✅ SUCCESS - Pipeline executed successfully!"
        }
        failure {
            echo "❌ FAILURE - Pipeline encountered an error!"
        }
    }
stage('Run Node Recovery Playbook') {
    steps {
        sh '''
            echo "=== RUNNING NODE RECOVERY PLAYBOOK ==="
            export PATH="$HOME/python/bin:$PATH"

            if [ -f "ansible/node_recovery.yml" ]; then
                echo "🚀 Running Ansible playbook: ansible/node_recovery.yml"
                ansible-playbook -i localhost, ansible/node_recovery.yml
            else
                echo "⚠️ Playbook not found"
            fi
        '''
    }
}

    
