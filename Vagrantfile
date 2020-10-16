# -*- mode: ruby -*-
# vi: set ft=ruby :

# This script to install Kubernetes will get executed after we have provisioned the box 
$common_script = <<-COMMONSCRIPT

  echo "********************************************************************************************"
  echo "COMMON ACTIONS"
  echo "********************************************************************************************"

  # Update apt package list and install HTTPS transport
  echo "*** Updating package list and installing apt-transport-https"
  apt-get update && apt-get install -y apt-transport-https

  # Get Google key for Kubernetes repo
  echo "*** Adding Google apt key and repo"
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
  deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

  # Update apt package list to include Google packages
  echo "*** Updating package list"
  apt-get update

  # Install kubelet, kubeadm and kubectl
  echo "*** Installing kubelet, kubeadm and kubectl"
  apt-get install -y kubelet kubeadm kubectl

  # kubelet requires swap off
  echo "*** Disabling swap"
  swapoff -a

  # keep swap off after reboot
  sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

  # Add cgroup driver and node IP arguments for kubelet
  echo "*** Adding cgroup and node IP args for kubelet..."
  EXT_IPADDR=`ifconfig eth1 | grep "inet " | awk '{print $2}'`
  sed -i "0,/ExecStart=/s//Environment=\\"KUBELET_EXTRA_ARGS=--node-ip=$EXT_IPADDR --cgroup-driver=cgroupfs\\"\\n&/" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

COMMONSCRIPT

$master_script = <<-MASTERSCRIPT

  echo "********************************************************************************************"
  echo "MASTER ACTIONS"
  echo "********************************************************************************************"

  # Get the IP addresses that VirtualBox has given this VM
  echo "*** Getting IP addresses"
  EXT_IPADDR=`ifconfig eth1 | grep "inet " | awk '{print $2}'`

  echo External IP $EXT_IPADDR
  echo $EXT_IPADDR > /vagrant/master-ip

  # Set up Kubernetes
  echo "*** Running kubeadm init"
  kubeadm init --node-name $(hostname -s) --apiserver-advertise-address=$EXT_IPADDR --pod-network-cidr=10.17.0.0/16 --service-cidr=10.18.0.0/24

  # Set up admin creds for the vagrant user
  echo "*** Copying credentials to /home/vagrant and setting ownership"
  sudo --user=vagrant mkdir -p /home/vagrant/.kube
  cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
  chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

  # Copy kubernetes admin.conf to shared /vagrant directory
  echo "*** Copying credentials to /vagrant"
  cp /etc/kubernetes/admin.conf /vagrant/

  # Install a network
  echo "*** Install weave pod network"
  kubectl --kubeconfig /vagrant/admin.conf apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl --kubeconfig /vagrant/admin.conf version | base64 | tr -d '\n')
  
  # Remove the taint on the master
  echo "*** Removing taint on master"
  kubectl --kubeconfig /vagrant/admin.conf taint nodes --all node-role.kubernetes.io/master-

  # Get the token required to join a worker to the cluster, write it to /vagrant/token
  echo "*** Saving join token to /vagrant/join-token"
  kubeadm token list -o yaml | grep token: | cut -d' ' -f2 > /vagrant/join-token

MASTERSCRIPT

$worker_script = <<-WORKERSCRIPT
  echo "********************************************************************************************"
  echo "WORKER ACTIONS"
  echo "********************************************************************************************"

  # Join the cluster
  echo "*** Running kubeadm join"
  kubeadm join $(cat /vagrant/master-ip | tr -d '\n'):6443 --token $(cat /vagrant/join-token | tr -d '\n') --discovery-token-unsafe-skip-ca-verification

  # Finally, echo the command to set up kubeconfig on the host machine
  echo "********************************************************************************************"
  echo ""
  echo "To use kubectl from your host, set the following env var (or use '--kubeconfig ./admin.conf'"
  echo ""
  echo '   export KUBECONFIG=$(pwd)/admin.conf'
  echo ""
  echo "********************************************************************************************"

WORKERSCRIPT

WORKERS = 2
CPU_MASTER = 2
CPU_WORKER = 2
MEMORY_MASTER = 2048
MEMORY_WORKER = 2048
LAB_NET = "10.8.8.0"
BOX_IMAGE = "bento/ubuntu-18.04"

class LabSetup

  def initialize
      p "################# K8S LAB SETUP #################"
  end

  def createMaster(config)
    masterIp = LAB_NET.split('.')[0..-2].join('.') + ".10"
    p "MASTER IP: #{masterIp}"

    config.vm.define "k8s-master" do |master|
      master.vm.box = BOX_IMAGE
      master.vm.hostname = "k8s-master"
      master.vm.network :private_network, ip: masterIp
      master.vm.provider "virtualbox" do |v|
        v.memory = MEMORY_MASTER
        v.customize ["modifyvm", :id, "--cpus", CPU_MASTER]
      end
      master.vm.provision "docker"
      master.vm.provision "shell", inline: $common_script
      master.vm.provision "shell", inline: $master_script
    end
  end

  def createWorkers(config)
    if WORKERS > 0
      (1..WORKERS).each do |i|
        workerIp = LAB_NET.split('.')[0..-2].join('.') + ".#{i + 10}"
        p "WORKER #{i} IP: #{workerIp}"

        config.vm.define "k8s-worker-#{i}" do |worker|
          worker.vm.box = BOX_IMAGE
          worker.vm.hostname = "k8s-worker-#{i}"
          worker.vm.network :private_network, ip: workerIp
          worker.vm.provider "virtualbox" do |v|
            v.memory = MEMORY_WORKER
            v.customize ["modifyvm", :id, "--cpus", CPU_WORKER]
          end
          worker.vm.provision "docker"
          worker.vm.provision "shell", inline: $common_script
          worker.vm.provision "shell", inline: $worker_script
        end
      end
    end
  end
end

Vagrant.configure("2") do |config|

  labSetup = LabSetup.new()

  labSetup.createMaster(config)
  labSetup.createWorkers(config)

end
