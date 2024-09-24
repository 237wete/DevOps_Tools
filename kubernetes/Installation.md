## INSTALLATION D'UN CLUSTER KUBERNETES 1.30

### 1. Environnement de travail
Nous allons effectuer notre installation avec des serveurs linux, notemment Rocky Linux 8.10

#### a. SANBOX

##### a.1. ACHITECTURE

![Capture d’écran du 2024-09-24 07-08-15](https://github.com/user-attachments/assets/69078970-75b5-4081-8264-9aba7ef44748)

#### a.2. Vagrant file d'installation

```markdown
# -*- mode: ruby -*-
# vi: set ft=ruby :
RAM = 2024
CPU = 2

Vagrant.configure("2") do |config|
 #Configuration de load balancer vagrant
  config.vm.define "k8s-control" do |k8s-control|
		k8s-control.vm.box = "geerlingguy/rockylinux8"
		k8s-control.vm.network "private_network", ip: "192.168.1.30" ,  netmask: "255.255.255.0" 
	end
  config.vm.provider "virtualbox" do |v|
		v.memory = RAM
		v.cpus = CPU
	end
 #Configuration de la premiere web machine
  config.vm.define "k8s-worker1" do |k8s-worker1|
		k8s-worker1.vm.box = "geerlingguy/rockylinux8"
		k8s-worker1.vm.network "private_network", ip: "192.168.1.31" ,  netmask: "255.255.255.0"
	end
  config.vm.provider "virtualbox" do |v|
		v.memory = RAM
		v.cpus = CPU
	end
 #Configuration de la seconde web machine
  config.vm.define "k8s-worker2" do |k8s-worker2|
		k8s-worker2.vm.box = "geerlingguy/rockylinux8"
		k8s-worker2.vm.network "private_network", ip: "192.168.1.32" ,  netmask: "255.255.255.0"
	end
  config.vm.provider "virtualbox" do |v|
		v.memory = RAM
		v.cpus = CPU
	end
end 

```
il faut au prealable avoir installer vagrant, si c'est pas encore le cas vous pouvez suivre le lien ci-dessous pour son installation https://developer.hashicorp.com/vagrant/docs/installation/ 

```
vagrant up
```

### a.2 Configuration du hostname et du fichier hosts

Sur le noeud Master (k8s-control) 

```
sudo hostnamectl set-hostname k8s-control
```
Sur le noeud worker1 (k8s-worker1)

```
sudo hostnamectl set-hostname worker1
```
Sur le noeud Worker2 (k8s-worker2) 

```
sudo hostnamectl set-hostname k8s-worker2
```

### a.3 Configuration du fichier hosts

Sur l'ensemble des nœuds, configurer le fichier hosts afin de faciliter la communication entre les noeuds via le nom de l'hôte.

```
sudo vi /etc/hosts
```
y ajouter les lignes ci-dessous:
```
192.168.1.30   k8s-control
192.168.1.31   k8s-worker1
192.168.1.32   k8s-worker2
```

### 2- Configuration de base sur tous les noeuds

**Mettons à jour selinux en mode permissive**

```
sudo setenforce 0
sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux
```

**Désactivation du swap**
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

**Installation de l'exécuteur de conteneur cri-o version 1.30**

Depuis la version 1.24 de K8S le conteneur runtime n'est plus docker mais cri-o. Nous allons l'installer des etapes ci-dessous

**Configuration du module kernel br_netfilter pour cri-o**

```
cat << EOF | sudo tee /etc/modules-load.d/cri-o.conf
br_netfilter
EOF

sudo modprobe br_netfilter
```

**Activation du routage des paquets et permission de la communication par pont entre conteneurs**
```
cat <<EOF | sudo tee /etc/sysctl.d/cri-o.conf
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

**Installation de container-selinux**

```
sudo dnf install -y container-selinux
```

**Installation de cri-o version 1.30**
```
sudo vi /etc/yum.repos.d/cri-o.repo
```
y ajouter les lignes ci-dessous:

```
[cri-o]
name=CRI-O
baseurl=https://pkgs.k8s.io/addons:/cri-o:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/addons:/cri-o:/stable:/v1.30/rpm/repodata/repomd.xml.key
exclude=cri-o
```
```
sudo dnf install -y cri-o --disableexcludes=cri-o
```
```
sudo systemctl start crio.service
sudo systemctl enable crio.service
```

### 3- Configuration du parefeu firewall-cmd pour autoriser les ports de k8s

**Sur le noeud master**

```
sudo systemctl start firewall-cmd
sudo firewall-cmd --permanent --add-port={6443/tcp,2379-2380/tcp,10250/tcp,10257/tcp,10259/tcp}
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --reload
```

**Sur les noeuds worker**
```
sudo systemctl start firewall-cmd
sudo firewall-cmd --permanent --add-port={10250/tcp,10256/tcp,30000-32767/tcp}
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --reload
```

Ces ports à autoriser se trouver sur le site officiel de K8s, nous pouvons y acceder via le lien: https://kubernetes.io/docs/reference/networking/ports-and-protocols/ 

### 4- Installation du cluster k8s

Ajout du repo k8s sur tous les noeuds
```
sudo vi /etc/yum.repos.d/kubernetes.repo
```
y ajouter les lignes ci-dessous

```
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
```
L'on pourra vérifier les versions disponibles pour 1.30 de chacun des packages kubelet, kubeadm et kubectl :

```
sudo dnf --showduplicates list kubelet --disableexcludes=kubernetes
sudo dnf --showduplicates list kubeadm --disableexcludes=kubernetes
sudo dnf --showduplicates list kubectl --disableexcludes=kubernetes
```

**Installation des packages kubelet, kubeadm et kubectl sur tous les noeuds**

```
sudo dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
verifions que cri-o  est bien installé via la commande:
```
sudo crictl image ls
```

**Démarrage et activation du kubelet sur tous les noeuds**
```
sudo systemctl enable --now kubelet
```

### Initialisation du cluster sur le noeud master

Sur le nœud master (k8s-control) uniquement, initialisons le cluster et configurons l'accès kubectl.

```
sudo kubeadm init --pod-network-cidr 172.16.0.0/16 --apiserver-advertise-address 192.168.1.2 --kubernetes-version 1.30.4
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**NB**: Ici 172.16.0.0/16 sera la plage du réseau privé de notre cluster et pour rappel l'ip 192.168.1.2 est celle de la master. 

**Installation du module complémentaire réseau Calico**
Ceci permettre la communication en nos noeuds, nous pouvons egalement utiliser flannel

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

**Ajout des noeuds worker au cluster**
L'on peut obtenir la commande de jointure avec la commande :

```
kubeadm token create --print-join-command
```

Le résultat de cette commande est également affichée à la fin de l'exécution de la commande kubeadm init.
Copiez la commande join (résultat de la commande ci-dessus) à partir du nœud master. Exécutez-le sur chaque nœud worker en tant que root (c'est-à-dire avec sudo ).

```
kubeadm join 192.168.1.30:6443 --token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Sur le nœud master, vérifiez que tous les nœuds de votre cluster sont prêts. Notez que cela peut prendre quelques instants pour que tous les nœuds passent à l'état PRÊT (READY).

```
kubectl get nodes
```

![Capture d’écran du 2024-09-24 08-10-47](https://github.com/user-attachments/assets/3d907896-e89f-4d65-b2f1-1a3914c8c8ed)


