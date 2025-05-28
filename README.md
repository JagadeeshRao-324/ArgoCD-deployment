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

     kubectl create namespace argocd

     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

     ##Expose the ArgoCD API server using port-forward, so we can start interacting with it:
     
     kubectl port-forward svc/argocd-server -n argocd 8080:443

     ##Install the ArgoCD CLI on the k8s cluster as it is a medium to interact with the ArgoCD API server

     VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
     
     curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/download/${VERSION}/argocd-linux-amd64

     chmod +x argocd

     sudo mv argocd /usr/local/bin/


- Login to the ArgoCD

      username: admin
  
      password: output of the below command

      kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

      argocd login localhost:8080 --username admin --password password --insecure


step3: Create the ArgoCD Application resource using CLI or YAML. it will create a ngnix deployment and a ngnix service.

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

Step4: Make changes in the ngnix yaml manifests. the changes will be automatically synced by the argoCD controller and reflected in the k8s environment

Current configuration:

           ##pod count = 1
           controlplane:~/ArgoCD-deployment$ kubectl get pods | grep -i ngnix
           ngnix-86d94f4645-s2vlk                              1/1     Running   0          5m8s

           ##image version-- ngnix:latest

  Example: change the number of replicas or the image version and commit the changes.

   - changed the replica count from 1 to 3
   - Image version from latest to 1.25.5

             git add .
             git commit -m "Updated the pod count and the image version"
             git push
     
            ##Pod count increased to 3
           controlplane:~$ kubectl get pods | grep -i ngnix
           ngnix-fc459ffbd-7ls4d                               1/1     Running   0          13s
           ngnix-fc459ffbd-bc28v                               1/1     Running   0          36s
           ngnix-fc459ffbd-f52vs                               1/1     Running   0          22s

           ##Image version changed from latest to 1.25.5
           Events:
              Type    Reason     Age   From               Message
              ----    ------     ----  ----               -------
              Normal  Scheduled  111s  default-scheduler  Successfully assigned argocd/ngnix-fc459ffbd-f52vs to node01
              Normal  Pulling    111s  kubelet            Pulling image "nginx:1.25.5"
