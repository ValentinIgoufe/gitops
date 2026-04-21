# Premier Projet Kubernetes

## Description :

> PPK = Initiation à k8s, ArgoCD et Terraform.

## Installation via Docker
### ./minikube start --driver=docker

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
## ingress.yaml
### .\tools\minikube.exe kubectl -- port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80

# Pour aller plus loin
## Ajouter un Load Balancer par dessous l'Ingress : user -> LB (cloud provider) -> Ingress -> Service -> Pods

# Refresh cluster
### .\tools\minikube.exe kubectl -- rollout restart deployment guestbook-ui -n mon-app