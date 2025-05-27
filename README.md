# ArgoCD-deployment
ArgoCD deployment

Prerequisites:
- Utilized the killer koda kubernetes playground for the k8s cluster
- ArgoCD as the GitOps controller

Implementation
Step1: Install ArgoCD on the K8s cluster

kubectl create namespace argocd
# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

![Image](https://github.com/user-attachments/assets/a47dad30-3e9a-42b1-92f4-a1322ff13c68)
