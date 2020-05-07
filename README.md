# Vagrant Kubernetes Setup
Within this repo is a Vagranfile that can be used to create a single node Kubernetes cluster in a VirtualBox VM, accessible from your host.

**Note: A static IP on a host-only network is specified in the Vagrantfile due to issues with the box not respecting DHCP IPs**

Actions performed:
* 


Install kubectl on your host:

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubectl
```  

Download the Vagrantfile and spin up your VM:

```vagrant up```



