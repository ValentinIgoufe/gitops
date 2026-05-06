# Premier Projet Kubernetes

## Description :

> PPK = Initiation à k8s, ArgoCD et Terraform.

## Installation via Docker
### ./minikube start --driver=docker --cni=calico --nodes 2 --memory=6144

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
# .\tools\minikube.exe kubectl -- -n monitoring port-forward svc/prometheus-operated 9090:9090
# .\tools\minikube.exe kubectl -- -n monitoring port-forward svc/monitoring-stack-grafana 3000:80
# .\tools\minikube.exe kubectl -- port-forward service/monitoring-stack-grafana -n monitoring 3000:80

# Right Sizing VPA

# Pour aller plus loin
## Ajouter un Load Balancer par dessous l'Ingress : user -> LB (cloud provider) -> Ingress -> Service -> Pods
## Une fois l'img push sur un registry -> on modifie le tag de l'img dans le repo GitOps pour trigger ArgoCD donc pas de :latest

# Refresh cluster
### .\tools\minikube.exe kubectl -- rollout restart deployment guestbook-backend-deployment -n guestbook-backend

# Admin ArgoCD
### kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
### xZ-fk24IpVriBbK-

# Longhorn
## ./tools/minikube ssh -n minikube "sudo apt-get update && sudo apt-get install -y open-iscsi && sudo systemctl enable --now iscsid"
## installer et activer open-iscsi


# EKS et Terraform
## https://www.periscop.tech/posts/20251127-terraform-kubernetes-eks-production/

# 1. Le Stockage (Persistent Volumes)
Dans K8s, par défaut, si un Pod meurt, ses données meurent avec lui. Pour de la prod, c'est inenvisageable. Tu dois comprendre :

PV / PVC (Persistent Volume Claims) : Comment on "réserve" 10 Go de disque.

StorageClasses : Comment K8s demande automatiquement au Cloud (AWS, Azure, ou ton propre serveur) de créer le disque.

CSI (Container Storage Interface) : Le standard qui permet à K8s de parler à n'importe quel disque.

À tester : Installe Longhorn ou OpenEBS sur ton petit cluster, c'est du stockage Open Source génial.

# 2. Le Service Mesh (La couche supérieure)
C'est la suite logique après avoir maîtrisé le réseau de base.

Istio ou Linkerd : Ça permet de sécuriser les communications entre tes Pods (mTLS) automatiquement, de gérer les "Retries" (réessayer si ça plante) et de voir tout le trafic en temps réel.

C'est le graal du DevSecOps pour le principe du Zero Trust.

# 3. L'Observabilité (Les yeux de l'admin)
Maîtriser K8s sans savoir ce qu'il s'y passe, c'est piloter un avion les yeux bandés.

Prometheus & Grafana : Pour les métriques (CPU, RAM, erreurs).

Loki / Fluentbit : Pour centraliser les logs.

Le plus : Apprendre à créer un Dashboard Grafana qui affiche l'état de santé de ton cluster.

# 4. La Sécurité (Runtime Security)
Toi qui aimes le SecOps, c'est là que tu vas t'éclater :

Falco : C'est le "système d'alarme". Il te prévient en temps réel si quelqu'un essaie d'ouvrir un shell dans un Pod en prod ou de lire un fichier sensible.

Trivy / Kyverno : Pour interdire le déploiement de Pods qui ont des failles de sécurité.

Après Prometheus/Grafana et EKS/Terraform, la suite logique ce serait :
Court terme (dans ta lancée)

RBAC — contrôle d'accès dans le cluster, qui peut faire quoi sur quelles ressources. C'est le least privilege appliqué aux utilisateurs/serviceaccounts, tu vas retrouver tes petits.
Secrets management — là t'as probablement des secrets en clair dans tes yamls ou des env vars. Intégrer Sealed Secrets ou External Secrets Operator avec un vault, c'est un vrai sujet en entreprise et ça boucle bien avec ton background sécu.

Moyen terme

Multi-environnements — gérer dev/staging/prod avec Helm values différentes ou Kustomize, c'est ce que t'as en prod en entreprise.
Service Mesh — Istio ou Linkerd, mTLS entre les pods, observabilité fine. C'est plus avancé mais ça impressionne.
Right Sizing VPA & Goldilocks