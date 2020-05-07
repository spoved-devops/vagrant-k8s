# Single-node or dual-node Kubernetes "Cluster" 
## Created in Virtualbox by Vagrant
Two Vagranfiles; one to create a single node cluster, the other to create a two node cluster.  Created in Virtualbox VM(s), with kubectl workable from your host.  Based on the ```bento/ubuntu-18.04``` box.

Docker is provisioned by Vagrant.  Inline scripts perform actions common to both master and worker, just the master, and just the worker.

**Note: static IP(s) on a host-only network are specified in the Vagrantfiles due to issues with the box not respecting DHCP IPs**

### Actions
* Vagrant:
  * Virtual Box private network created
  * One or two Virtualbox VMs created, depending on Vagrantfile used:
    * k8s-master, static IP 172.28.128.100
    * k8s-worker, static IP 172.28.128.101
  * Current directory on the host mapped to /vagrant in the VM(s)
  * Docker provisioned
* Inline scripts:
  * Common:
    * Package list updated
    * ```apt-transport-https``` installed
    * Google key downloaded and added using ```apt-key add```
    * Package list updated (again)
    * Packages installed: ```kubelet```, ```kubeadm``` and ```kubeadm```
    * Swap turned off using ```swap -a```
    * Persist swap turn off in ```/etc/fstab```
    * Add ```-cgroup-driver=cgroupfs``` argument to ```kubeadm``` boot script
  * Master:
    * Grab internal and external IPs
    * Run ```kubeadm init``` specifying use of the "external" IP and setting a CIDR range for the pod network
    * Copy ```/etc/kubernetes/admin.conf``` to ```/home/vagrant/.kube/config``` to allow local access to the cluster by the ```vagrant``` user
    * Copy ```/etc/kubernetes/admin.conf``` to ```/vagrant/admin.conf``` - this makes the API server credentials file available on the host
    * Install the weave network on the cluster
    * Remove the default taint preventing workloads on the master node (since this is a single node cluster)
    * Save the join token needed on the worker to ```/vagrant/join-token```
  * Worker
    * Run ```kubeadm join``` using the join token available in ```/vagrant```

### Usage
Ensure Vagrant and Virtualbox are installed.

Install kubectl on your host if you want to be able to manage the cluster from outside your VM:

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubectl
```  

Clone the repo and rename the preferred Vagrantfile:

```bash
git clone https://github.com/spoved-devops/vagrant-k8s.git
cp Vagrantfile-1-node Vagrantfile
```

Spin things up:
```bash
vagrant up
```

Export the KUBECONFIG environment variable shown as the final output from ```vagrant up```

```bash
export KUBECONFIG=$(pwd)/admin.conf
```

The worker node may take a couple of minutes to become fully ready after joining the cluster.  Check with:
```bash
kubectl get nodes
```

## Issues
* Occasionally the provisioning of docker-ce by Vagrant will fail, with a mix of potential errors all seemingly related to issues downloading either packages or verifying certificates