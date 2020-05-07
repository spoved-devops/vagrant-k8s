# -*- mode: ruby -*-
# vi: set ft=ruby :

# This script to install Kubernetes will get executed after we have provisioned the box 
$script = <<-SCRIPT

  # Update apt package list and install HTTPS transport
  apt-get update && apt-get install -y apt-transport-https

  # Get Google key for Kubernetes repo
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
  deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

  # Update apt package list to include Google packages
  apt-get update

  # Install kubelet, kubeadm and kubectl
  apt-get install -y kubelet kubeadm kubectl

  # kubelet requires swap off
  swapoff -a

  # keep swap off after reboot
  sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
  sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

  # Get the IP addresses that VirtualBox has given this VM
  INT_IPADDR=`ifconfig eth0 | grep "inet " | awk '{print $2}'`
  EXT_IPADDR=`ifconfig eth1 | grep "inet " | awk '{print $2}'`

  echo Internal IP $INT_IPADDR
  echo External IP $EXT_IPADDR

  # Set up Kubernetes
  NODENAME=$(hostname -s)
  kubeadm init --apiserver-cert-extra-sans=$EXT_IPADDR --node-name $NODENAME  # --apiserver-advertise-address=$EXT_IPADDR

  # Set up admin creds for the vagrant user
  echo Copying credentials to /home/vagrant...
  sudo --user=vagrant mkdir -p /home/vagrant/.kube
  cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
  chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

  # Copy kubernetes admin.conf to shared /vagrant directory
  cp /etc/kubernetes/admin.conf /vagrant/

  # Install a network
  kubectl --kubeconfig /vagrant/admin.conf apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl --kubeconfig /vagrant/admin.conf version | base64 | tr -d '\n')

  # Remove the taint on the master
  kubectl --kubeconfig /vagrant/admin.conf taint nodes --all node-role.kubernetes.io/master-

  # Replace internal IP with external IP in the config copied for use on the host
  sed -i -e "s/${INT_IPADDR}/${EXT_IPADDR}/" /vagrant/admin.conf

  # Finally, echo the command to set up kubeconfig on the host machine
  echo "********************************************************************************************"
  echo
  echo "To use kubectl from your host, set the following env var (or use '--kubeconfig ./admin.conf'"
  echo 
  echo '   KUBECONFIG=$(pwd)/admin.conf'
  echo
  echo "********************************************************************************************"
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end
  
  # Set hostname, image and network type
  config.vm.hostname = "k8s-test1"
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.network "private_network", ip: "172.28.128.99"
  config.vm.define "k8s-test"
  
  # Install docker on the VM
  config.vm.provision "docker"

  # Run the inline script  
  config.vm.provision "shell", inline: $script
end
