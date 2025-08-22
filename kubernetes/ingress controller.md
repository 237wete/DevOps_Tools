## INTALLATION DE L' INGRESS CONTROLLEUR
=====================================

DOC: https://kubernetes.github.io/ingress-nginx/deploy/

Documentation officiel
======================

https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/

Nous allons installer en utilisant helm

1- ajouter le depos de ingress-nginx 

   NB: ne pas être en sudo

   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   
2- pour vérifier les repos 

   helm search repo ingress-nginx --versions
   
3- pour savoir la version de l'ingress qu'il faut installer nous allons aller sur le site officiel

   http://github.com/kubernetes/ingress-nginx
   
   puisque nous utilisaons la version 1.29 de kubernetes, nous allons prendre la v1.10.0 de l'ingress controller
   ```
   Créer le repertoire ./k8s/ingress-nginx/
    mkdir -r ./k8s/ingress-nginx/
    ls ./k8s/ingress-nginx/
   ```
   ```
   helm template ingress-nginx ingress-nginx \
   --repo https://kubernetes.github.io/ingress-nginx \
   --version 4.10.0 \
   --namespace ingress-nginx \
   > ./k8s/ingress-nginx/ingress-nginx-1.10.0.yaml
   ```
   Sudo su pour la creation du ns
  ``` 
   Sudo su
   ```
   ```
   kubectl create ns ingress-nginx
   ```
   Installation de l'ingress controller
   ```
   kubectl apply -f ./k8s/ingress-nginx/ingress-nginx-1.10.0.yaml
   ```
   ```
   kubectl get all -n ingress-nginx
   ```
