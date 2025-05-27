# ArgoCD-deployment
ArgoCD deployment

Prerequisites:
- Utilized the killer koda kubernetes playground for the k8s cluster
- ArgoCD as the GitOps controller

Implementation

step1: Fork the git repo containing the neccessary yaml manifests to deploy the app to the local kubernetes node
      
      # Clone your fork
      
      git clone https://github.com/JagadeeshRao-324/ArgoCD-deployment.git
      
      cd ArgoCD-deployment

Step2: Install ArgoCD on the K8s cluster

- kubectl create namespace argocd
# Install ArgoCD
- kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
- Login to the ArgoCD
      username: admin
      password: output of the below command

      kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

      argocd login localhost:8080 --username admin --password password --insecure


step3: Create the ArgoCD Application resource using CLI or YAML.

   CLI:

       argocd app create nginx-app \
       --repo https://github.com/JagadeeshRao-324/ArgoCD-deployment.git \
       --path . \
       --dest-server https://kubernetes.default.svc \
       --dest-namespace argocd \
       --sync-policy automated
   
   YAML:

       # nginx-app.yaml
      apiVersion: argoproj.io/v1alpha1
      kind: Application
      metadata:
         name: nginx-app
         namespace: argocd
      spec:
         project: default
         source:
           repoURL: https://github.com/JagadeeshRao-324/ArgoCD-deployment.git
           targetRevision: HEAD
           path: nginx-app
         destination:
           server: https://kubernetes.default.svc
           namespace: argocd
         syncPolicy:
           automated:
             selfHeal: true
             prune: true

        kubectl apply -f nginx-app.yaml

