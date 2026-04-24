# Premier Projet Kubernetes

## Description :

> PPK = Initiation à k8s, ArgoCD et Terraform.

## Installation via Docker
### ./minikube start --driver=docker --cni=calico --nodes 3

# Control Plane OK ?
### ./minikube.exe kubectl -- get nodes

# Dashboard 
### .\minikube.exe dashboard

# ArgoCD 

## Namespace
### .\minikube.exe kubectl -- create namespace argocd

## Installation 
### kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

## ArgoCD OK ?
### .\minikube.exe kubectl -- get pods -n argocd

## Link avec un repo
### .\minikube.exe kubectl -- apply -f guestbook-app.yaml

## App OK ?
### .\minikube.exe kubectl -- get pods -n default

## Accès à l'app
### .\minikube.exe service guestbook-ui -n default

## Déplacer de default vers un autre namespace
> créé le namespace, changer la destination dans le yaml et apply
### Obersavtion : 
> ArgoCD a supprimé celui dans default, pq ? Grâce à SyncPolicy : prune true. selfHeal permet de sync uniquement avec le yaml, on ne peut pas créer à la main un autre prod dans un autre namespace l'app en question.

# Passer à 3 nodes
### .\minikube.exe delete
### .\minikube.exe start --driver=docker --nodes 3
### .\minikube.exe kubectl -- apply -f guestbook-app.yaml
### ./minikube.exe kubectl -- get nodes -> OK
### .\minikube.exe kubectl -- create namespace mon-app + argocd
### .\minikube.exe kubectl -- config set-context --current --namespace=argocd 
### .\minikube.exe kubectl -- apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
### .\minikube.exe kubectl -- apply -f guestbook-app.yaml

# HPA
### hpa.yaml
### .\minikube.exe addons enable metrics-server
### .\minikube.exe kubectl -- top pods -n mon-app

# Stress Tests pour HPA 
## .\tools\minikube.exe kubectl -- exec -it guestbook-ui-f8bf55fbf-xpnl8 -n mon-app -- /bin/sh -c "while true; do :; done"

# Ingress (Reverse Proxy)
## addons enable ingress
## ingress.yaml
### .\tools\minikube.exe kubectl -- port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80

# Least Privilege
### deployment.yaml

# Network Policies
## start avec calico 
### tester les règles : kubectl run test-terminal --rm -it --image=alpine -- sh 
### + wget -qO- --timeout=2 http://guestbook-backend-service.guestbook-backend.svc.cluster.local
### + wget -qO- --timeout=2 http://guestbook-frontend-service.guestbook-frontend.svc.cluster.local
### get pods -n kube-system | findstr calico

# Helm
## changement de structure -> Chart Umbrella 
### .\tools\minikube.exe kubectl -- apply -f .\cluster-config\argocd\guestbook\guestbook-helm.yaml

# Monitoring
## Prometheus & Grafana & AlertManager
### Add dependancies in Chart.yaml and some configs.yaml in the parent

# Pour aller plus loin
## Ajouter un Load Balancer par dessous l'Ingress : user -> LB (cloud provider) -> Ingress -> Service -> Pods
## Une fois l'img push sur un registry -> on modifie le tag de l'img dans le repo GitOps pour trigger ArgoCD donc pas de :latest

# Refresh cluster
### .\tools\minikube.exe kubectl -- rollout restart deployment guestbook-backend-deployment -n guestbook-backend

# Admin ArgoCD
### d-XpIu7e3MfgHTy3