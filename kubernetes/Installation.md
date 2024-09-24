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


