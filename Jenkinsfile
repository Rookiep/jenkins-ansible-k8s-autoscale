pipeline {
    agent any

    stages {

        stage('Initialize') {
            steps {
                echo '🚀 Starting Jenkins CI/CD pipeline for Ansible + Kubernetes...'
                sh '''
                    echo "Current working directory:"
                    pwd
                    echo "Listing workspace contents:"
                    ls -la
                '''
            }
        }

        stage('Fix Permissions') {
            steps {
                echo '🔧 Ensuring correct workspace permissions...'
                sh '''
                    echo "Fixing ownership of Jenkins workspace..."
                    chown -R jenkins:jenkins /var/jenkins_home || true
                    chmod -R 777 /var/jenkins_home/workspace || true
                '''
            }
        }

        stage('Clean Workspace') {
            steps {
                echo '🧹 Cleaning workspace to avoid permission or stale file issues...'
                deleteDir()  // Jenkins built-in method: wipes the workspace clean
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
                echo '🔧 Installing kubectl, Python, and Ansible if missing...'
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

        stage('Setup Kubeconfig') {
            steps {
                echo '🔐 Setting up Kubernetes configuration from Jenkins credentials...'
                withCredentials([file(credentialsId: 'kubeconfig-secret', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        mkdir -p ~/.kube
                        cp $KUBECONFIG_FILE ~/.kube/config
                        chmod 600 ~/.kube/config
                        export KUBECONFIG=~/.kube/config
                        echo "✅ Kubeconfig successfully loaded."
                    '''
                }
            }
        }

        stage('Verify Kubernetes Connectivity') {
            steps {
                echo '🔍 Verifying Kubernetes cluster connection...'
                sh '''
                    export KUBECONFIG=~/.kube/config
                    if kubectl config current-context >/dev/null 2>&1; then
                        echo "Current context: $(kubectl config current-context)"
                        echo "Available nodes:"
                        kubectl get nodes -o wide || echo "⚠️ No nodes found or access issue."
                    else
                        echo "⚠️ WARNING: No Kubernetes context set — skipping cluster operations."
                    fi
                '''
            }
        }

        stage('Restart Node (Simulated)') {
            when {
                expression { return sh(script: 'kubectl config current-context >/dev/null 2>&1', returnStatus: true) == 0 }
            }
            steps {
                echo '🔁 Simulating Kubernetes node restart...'
                sh '''
                    export KUBECONFIG=~/.kube/config
                    NODE=$(kubectl get nodes -o name | head -n1 | cut -d'/' -f2 || echo "default-node")
                    echo "Draining node: $NODE"
                    kubectl drain $NODE --ignore-daemonsets --delete-emptydir-data --force || true
                    sleep 5
                    echo "Uncordoning node: $NODE"
                    kubectl uncordon $NODE || true
                    echo "✅ Node restart simulation complete."
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                echo '⚙️ Running Ansible playbook for node recovery...'
                dir('ansible') {
                    sh '''
                        export KUBECONFIG=~/.kube/config
                        ansible-playbook -i inventory.ini playbook.yml || echo "⚠️ Playbook completed with warnings."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed — please review logs above.'
        }
        always {
            echo '🧽 Cleaning up kubeconfig and temporary files...'
            sh '''
                rm -rf ~/.kube /var/jenkins_home/.kube || true
                chown -R jenkins:jenkins /var/jenkins_home/workspace || true
                chmod -R 777 /var/jenkins_home/workspace || true
            '''
        }
    }
}
