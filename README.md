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
### Important: Use these syncPolicy options to avoid blocking with admission webhooks:
### - Replace=true (not ServerSideApply)
### - FailOnSharedResource=false
### - prunePropagationPolicy: foreground
### If stuck with namespace Terminating: delete webhooks first before deleting Application
### kubectl delete validatingwebhookconfigurations -l app.kubernetes.io/name=prometheus
### kubectl delete mutatingwebhookconfigurations -l app.kubernetes.io/name=prometheus

# Pour aller plus loin
## Ajouter un Load Balancer par dessous l'Ingress : user -> LB (cloud provider) -> Ingress -> Service -> Pods
## Une fois l'img push sur un registry -> on modifie le tag de l'img dans le repo GitOps pour trigger ArgoCD donc pas de :latest

# Refresh cluster
### .\tools\minikube.exe kubectl -- rollout restart deployment guestbook-backend-deployment -n guestbook-backend

# Admin ArgoCD
### SlplTErwCcbf1XTb

.\tools\minikube.exe kubectl -- patch rolebinding monitoring-stack-prometheu-admission -p '{\"metadata\":{\"finalizers\":[]}}' --type=merge -n monitoring
.\tools\minikube.exe kubectl -- delete rolebinding monitoring-stack-prometheu-admission --grace-period=0 --force -n monitoring           

kubectl delete clusterrole monitoring-stack-prometheu-admission
kubectl delete clusterrolebinding monitoring-stack-prometheu-admission
kubectl delete role monitoring-stack-prometheu-admission -n monitoring
kubectl delete rolebinding monitoring-stack-prometheu-admission -n monitoring


#
# REMETTRE LES NETWORK POLICIES ET LES ALERTES PROMETHEUS
#


# .\tools\minikube.exe kubectl -- -n monitoring port-forward svc/prometheus-operated 9090:9090
# .\tools\minikube.exe kubectl -- -n monitoring port-forward svc/monitoring-stack-grafana 3000:80

Après Prometheus/Grafana et EKS/Terraform, la suite logique ce serait :
Court terme (dans ta lancée)

RBAC — contrôle d'accès dans le cluster, qui peut faire quoi sur quelles ressources. C'est le least privilege appliqué aux utilisateurs/serviceaccounts, tu vas retrouver tes petits.
Secrets management — là t'as probablement des secrets en clair dans tes yamls ou des env vars. Intégrer Sealed Secrets ou External Secrets Operator avec un vault, c'est un vrai sujet en entreprise et ça boucle bien avec ton background sécu.

Moyen terme

Multi-environnements — gérer dev/staging/prod avec Helm values différentes ou Kustomize, c'est ce que t'as en prod en entreprise.
Service Mesh — Istio ou Linkerd, mTLS entre les pods, observabilité fine. C'est plus avancé mais ça impressionne.