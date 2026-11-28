<img width="1900" height="967" alt="Argocd_UI" src="https://github.com/user-attachments/assets/0913baa2-84e3-4244-b754-e61ee2cc101a" />
<img width="1035" height="651" alt="VM_k8s_app" src="https://github.com/user-attachments/assets/11186285-d46b-406c-b45c-f9ebeb867e34" />

**ArgoCD + Argo Rollouts Canary Deployment on Kubernetes
**Overview
**This project demonstrates a complete GitOps-based deployment workflow using:
**
**Kubernetes (Kubeadm Cluster)
Containerd Runtime
Calico CNI
ArgoCD
Argo Rollouts
GitHub
Canary Deployment Strategy
Architecture**
GitHub Repository
        │
        ▼
     ArgoCD
        │
        ▼
  Argo Rollouts
        │
        ▼
 Canary Deployment
        │
        ▼
   Kubernetes Cluster
**Prerequisites**
Infrastructure
Component	Quantity
Master Node	1
Worker Nodes	2
OS	Ubuntu 22.04+
Container Runtime	Containerd
CNI	Calico

**Kubernetes Cluster Setup**
**Master Node
**
Install Containerd
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installContainerd.sh -P /tmp

sudo bash /tmp/installContainerd.sh

sudo systemctl restart containerd.service
Install Kubernetes
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp

sudo bash /tmp/installK8S.sh
Initialize Cluster
sudo kubeadm init --ignore-preflight-errors=all
Configure Kubectl
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
Install Calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico.yaml
Generate Join Command
kubeadm token create --print-join-command

**Worker Nodes**
Install Containerd
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installContainerd.sh -P /tmp

sudo bash /tmp/installContainerd.sh

sudo systemctl restart containerd.service
Install Kubernetes
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp

sudo bash /tmp/installK8S.sh
Join Cluster
Run the join command generated from the master node:

kubeadm join <MASTER-IP>:6443 --token <TOKEN> \
--discovery-token-ca-cert-hash sha256:<HASH>

**Verify Cluster**
kubectl get nodes
kubectl get pods -A

**Install ArgoCD**
Create Namespace
kubectl create namespace argocd
Install ArgoCD
kubectl apply --server-side -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Verify
kubectl get pods -n argocd
Expose ArgoCD UI
kubectl patch svc argocd-server -n argocd \
-p '{"spec":{"type":"NodePort"}}'
Get Service Details
kubectl get svc -n argocd
Get ArgoCD Admin Password
kubectl get secret argocd-initial-admin-secret -n argocd \
-o jsonpath="{.data.password}" | base64 -d && echo
Default Username
admin

**Install ArgoCD CLI**
curl -sSL -o argocd \
https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

chmod +x argocd

sudo mv argocd /usr/local/bin/
Verify
argocd version
Install Helm
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
Verify
helm version

**Install Argo Rollouts**
Add Helm Repository
helm repo add argo https://argoproj.github.io/argo-helm

helm repo update
Create Namespace
kubectl create namespace argo-rollouts
Install Rollouts
helm install argo-rollouts argo/argo-rollouts \
-n argo-rollouts
Verify Argo Rollouts
kubectl get pods -n argo-rollouts
kubectl get crd | grep rollouts
kubectl api-resources | grep rollout
Expected Output:

rollouts.argoproj.io
rollouts   ro   argoproj.io/v1alpha1
Install Argo Rollouts CLI
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64

chmod +x kubectl-argo-rollouts-linux-amd64

sudo mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
Verify
kubectl argo rollouts version

**Rollout Example**
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: kubeserve-canary
spec:
  replicas: 4

  selector:
    matchLabels:
      app: kubeserve

  template:
    metadata:
      labels:
        app: kubeserve
    spec:
      containers:
      - name: c1
        image: leaddevops/kubeserve:v3
        ports:
        - containerPort: 80

  strategy:
    canary:
      steps:
      - setWeight: 25
      - pause:
          duration: 30s

      - setWeight: 50
      - pause:
          duration: 30s

      - setWeight: 75
      - pause:
          duration: 30s

      - setWeight: 100
      
**Create ArgoCD Application**
Field	Value
Application Name	canary-app
Project	default
Repository URL	GitHub Repository
Revision	HEAD
Path	.
Cluster URL	https://kubernetes.default.svc
Namespace	default
Sync Application
Create
   ↓
Refresh
   ↓
Sync
Monitor Rollout
Check Rollouts
kubectl get rollout
Watch Rollout
kubectl argo rollouts get rollout kubeserve-canary --watch
Example Healthy Output
Status: Healthy

SetWeight: 100

Replicas:
Desired: 4
Ready: 4
Available: 4
Rollback Strategy
GitOps Rollback (Recommended)
Change image:

image: leaddevops/kubeserve:v3
Back to:

image: leaddevops/kubeserve:v2
Commit and push:

git add .

git commit -m "Rollback to v2"

git push origin main
ArgoCD automatically syncs and Argo Rollouts performs rollback.

Troubleshooting
Node Status
kubectl get nodes
Cluster Pods
kubectl get pods -A
Node Details
kubectl describe node master

kubectl describe node worker1

kubectl describe node worker2
Disk Usage
df -h
Kubelet Status
systemctl status kubelet
Containerd Status
systemctl status containerd
Learning Outcomes
Kubernetes Cluster Setup using Kubeadm
Containerd Runtime Configuration
Calico CNI Installation
ArgoCD Installation and Configuration
GitOps Workflow Implementation
Argo Rollouts Installation
Canary Deployment Strategy
Automated Rollback Process
Progressive Traffic Shifting
Kubernetes Troubleshooting
Author
Ankit Kothawade

DevOps | Kubernetes | ArgoCD | Argo Rollouts | GitOps
