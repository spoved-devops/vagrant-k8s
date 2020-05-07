# Single-node Kubernetes "Cluster" 
## Created in Virtualbox by Vagrant
A Vagranfile that can be used to create a single node Kubernetes cluster in a VirtualBox VM, accessible from your host.  Based on the ```bento/ubuntu-18.04``` box.

**Note: A static IP on a host-only network is specified in the Vagrantfile due to issues with the box not respecting DHCP IPs**

### Actions
* Vagrant:
  * Virtual Box VM named k8s-test created
  * Virtual Box private network created
  * Static IP assigned - 172.28.128.99
  * Docker provisioned
  * Current directory on the host mapped to /vagrant in the VM (default behaviour)
* Inline script:
  * Package list updated
  * ```apt-transport-https``` installed
  * Google key downloaded and added using ```apt-key add```
  * Package list updated (again)
  * Packages installed: ```kubelet```, ```kubeadm``` and ```kubeadm```
  * Swap turned off using ```swap -a```
  * Persist swap turn off in ```/etc/fstab```
  * Add ```-cgroup-driver=cgroupfs``` argument to ```kubeadm``` boot script
  * Grab the internal and external IPs
  * Run ```kubeadm init``` to create the k8s cluster, specifying the external IP as an additional name for the API server certificate - this allows ```kubectl``` access from the host using the host-networking IP address
  * Copy ```/etc/kubernetes/admin.conf``` to ```/home/vagrant/.kube/config``` to allow local access to the cluster by the ```vagrant``` user
  * Copy ```/etc/kubernetes/admin.conf``` to ```/vagrant/admin.conf``` - this makes the API server credentials file available on the host
  * Install the weave network on the cluster
  * Remove the default taint preventing workloads on the master node (since this is a single node cluster)
  * Replace the internal IP in ```/vagrant/admin.conf``` with the host-networking IP
  * Change ownership of ```/home/vagrant/.kube/config``` to the ```vagrant``` user


### To Use:
Ensure Vagrant and Virtualbox are installed.

Install kubectl on your host if you want to be able to manage the cluster from outside your VM:

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubectl
```  

Clone the repo and spin things up:

```bash
git clone https://github.com/spoved-devops/vagrant-k8s.git
vagrant up
```

Export the KUBECONFIG environment variable shown as the final output from ```vagrant up```



