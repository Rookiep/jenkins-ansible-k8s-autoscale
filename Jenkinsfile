pipeline {
    agent {
        docker {
            image 'ubuntu:20.04'
            args '--privileged --network=host -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Setup Environment') {
            steps {
                sh '''
                apt-get update
                apt-get install -y software-properties-common
                add-apt-repository --yes --update ppa:ansible/ansible
                apt-get install -y ansible curl
                
                # Install kubectl
                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
                
                # Copy kubeconfig from host (if Jenkins has access)
                mkdir -p /root/.kube
                cp /home/jenkins/.kube/config /root/.kube/config || true
                '''
            }
        }
        
        stage('Create Test Failure') {
            steps {
                sh '''
                # Make a node NotReady for testing
                minikube ssh "sudo systemctl stop kubelet" || echo "Could not stop kubelet"
                
                # Wait a bit for status change
                sleep 30
                '''
            }
        }
        
        stage('Run Ansible Recovery') {
            steps {
                sh '''
                git clone https://github.com/Rookiep/jenkins-ansible-k8s-autoscale.git .
                ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
                '''
            }
        }
    }
}
