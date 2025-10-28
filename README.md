# jenkins-ansible-k8s-autoscale

Ready-to-import example repo for a local Jenkins + Ansible + Kubernetes CI/CD pipeline.

## Contents
- Jenkinsfile : Jenkins pipeline (checks out repo, deploys k8s, runs Ansible playbook, applies HPA)
- ansible/ : inventory and node recovery playbook (minikube-focused)
- k8s/ : deployment, service, and HPA manifests
- .gitignore

## Quick start (assumes Minikube)
1. Unzip and open in VS Code.
2. Start Minikube:
   ```bash
   minikube start --cpus=4 --memory=8192
   ```
3. Install metrics-server for HPA:
   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   kubectl patch deployment metrics-server -n kube-system --type=json -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
   ```
4. Run Jenkins (optional) via Docker (or use local Jenkins):
   ```bash
   docker run -d -p 8080:8080 -p 50000:50000 \
     -v jenkins_home:/var/jenkins_home \
     -v /var/run/docker.sock:/var/run/docker.sock \
     --name jenkins jenkins/jenkins:lts
   ```
5. Create a Jenkins pipeline pointing to this repo's Jenkinsfile (or run stages locally).
6. Deploy manually (for quick test):
   ```bash
   kubectl apply -f k8s/deployment.yaml
   kubectl apply -f k8s/service.yaml
   kubectl apply -f k8s/hpa.yaml
   ```
7. Test Ansible node recovery:
   - Simulate NotReady:
     ```bash
     kubectl taint nodes $(kubectl get nodes -o name | cut -d'/' -f2) node.kubernetes.io/unreachable=true:NoSchedule
     ```
   - Run:
     ```bash
     ansible-playbook -i ansible/inventory.ini ansible/node_recovery.yml
     ```

## Note
This ZIP assumes **Minikube** as the local Kubernetes environment and includes Ansible commands that restart minikube nodes. If you use Docker Desktop Kubernetes, some commands will need adjusting.
