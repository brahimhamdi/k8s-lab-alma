Vagrant.require_version ">= 2.0.0"

boxes = [
    {
        :name => "kube-control-plane-alma",
        :eth1 => "192.168.56.110",
        :mem => "4096",
        :cpu => "2"
    },
    {
        :name => "kube-node1-alma",
        :eth1 => "192.168.56.111",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "kube-node2-alma",
        :eth1 => "192.168.56.112",
        :mem => "1024",
        :cpu => "1"
    }
]

Vagrant.configure(2) do |config|
  config.vm.box = "almalinux/9"

  config.vbguest.auto_update = false if Vagrant.has_plugin?("vagrant-vbguest")

  boxes.each do |opts|
      config.vm.define opts[:name] do |config|
        config.vm.hostname = opts[:name]
        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", opts[:mem]]
          v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
          v.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          v.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
        end
        config.vm.network :private_network, ip: opts[:eth1]
      end
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo setenforce 0
    sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux
    sestatus
    sudo firewall-cmd --permanent --add-port=6443/tcp
    sudo firewall-cmd --permanent --add-port=2379-2380/tcp
    sudo firewall-cmd --permanent --add-port=10250/tcp
    sudo firewall-cmd --permanent --add-port=10251/tcp
    sudo firewall-cmd --permanent --add-port=10259/tcp
    sudo firewall-cmd --permanent --add-port=10257/tcp
    sudo firewall-cmd --permanent --add-port=179/tcp
    sudo firewall-cmd --permanent --add-port=4789/udp
    sudo firewall-cmd --permanent --add-port=30000-32767/tcp
    sudo firewall-cmd --permanent --add-port=4789/udp
    sudo firewall-cmd --reload
    sudo yum update
    sudo yum install -y yum-utils
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    sudo yum install -y containerd.io --allowerasing

    sudo modprobe overlay
    sudo modprobe br_netfilter
    sudo echo 'overlay' > /etc/modules-load.d/containerd.conf
    sudo echo 'br_netfilter' >> /etc/modules-load.d/containerd.conf
    sudo echo 'net.bridge.bridge-nf-call-iptables = 1' > /etc/sysctl.d/kubernetes.conf
    sudo echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.d/kubernetes.conf

    sudo echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.d/kubernetes.conf
    sudo sysctl --system
    echo "KUBELET_EXTRA_ARGS=--node-ip="$(ip addr show eth1  | awk '$1 == "inet" { print $2 }' | cut -d/ -f1) | sudo tee /etc/default/kubelet
    sudo containerd config default | sudo tee /etc/containerd/config.toml
    sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
    sudo systemctl restart containerd

    sudo swapoff -a
    sudo sed -i '/ swap / s/^/#/' /etc/fstab 
 
  SHELL

end
