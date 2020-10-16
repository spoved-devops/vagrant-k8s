# Multi-node Kubernetes Cluster
Created by Vagrant as Virtualbox VMs, with kubectl workable from your host.  Based on the ```bento/ubuntu-18.04``` box.

Docker is provisioned by Vagrant.  Inline scripts perform actions common to both master and worker, just the master, and just the worker.

### Actions
* Vagrant:
  * Virtual Box private network created
  * One master created
  * Zero or more workers created
  * Current directory on the host mapped to /vagrant in the VM(s)
  * Docker provisioned
* Inline scripts:
  * Common:
    * Package list updated, ```apt-transport-https``` installed
    * Google repo added, key downloaded and added using ```apt-key add```
    * Package list updated (again)
    * Packages installed: ```kubelet```, ```kubeadm``` and ```kubeadm```
    * Swap turned off using ```swap -a```
    * Persist swap turn off in ```/etc/fstab```
    * Add ```--cgroup-driver=cgroupfs``` and ```--node-ip=<external ip>``` arguments to ```kubeadm``` boot script
  * Master:
    * Grab external IP and write to ```/vagrant/master-ip``` - used when configuring additional worker node
    * Run ```kubeadm init``` specifying use of the "external" IP and setting a pod and service CIDR ranges 
    * Copy ```/etc/kubernetes/admin.conf``` to ```/home/vagrant/.kube/config``` to allow local access to the cluster by the ```vagrant``` user
    * Copy ```/etc/kubernetes/admin.conf``` to ```/vagrant/admin.conf``` - this makes the API server credentials file available on the host
    * Install the weave pod network on the cluster
    * Remove the default taint preventing workloads on the master node (since this is a single node cluster)
    * Save the join token needed on the worker to ```/vagrant/join-token```
  * Worker
    * Run ```kubeadm join``` using the join token available in ```/vagrant```

### Usage
Ensure Vagrant and Virtualbox are installed.

Install kubectl on your host if you want to be able to manage the cluster from outside your VM(s):

```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl

```  

Clone the repo and rename the preferred Vagrantfile:

```bash
git clone https://github.com/spoved-devops/vagrant-k8s.git
```

Spin things up:
```bash
vagrant up
```

Export the KUBECONFIG environment variable shown as the final output from ```vagrant up```

```bash
export KUBECONFIG=$(pwd)/admin.conf
```

The worker nodes may take a couple of minutes to become fully ready after joining the cluster.  Check with:
```bash
kubectl get nodes
```
