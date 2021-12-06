
# -*- mode: ruby -*-
# vi: set ft=ruby :
# vagrant plugin install vagrant-vbguest
# vagrant plugin install vagrant-disksize

#     subconfig.vm.network "private_network", ip: "10.0.0.101"
#      # subconfig.vm.network "forwarded_port", id: "ssh", host: 2101, guest: 22
#      # subconfig.vm.network "forwarded_port", guest:80, host:8080
#      # subconfig.vm.network "forwarded_port", guest:443, host:8443
   
#    #  subconfig.vm.synced_folder "ansible", "/root/ansible", create: true, owner: "root", group: "root"




BOX_DEBIAN = "debian/buster64"
BOX_UBUNTU_DESKTOP = "peru/ubuntu-20.04-desktop-amd64"



$ansible = <<-SHELL
  sudo apt install git python3-pip -y
  sudo pip3 install ansible -y
  sudo mkdir /root/ansible
  sudo touch /root/ansible/hosts
  sudo touch /root/ansible.cfg
SHELL

$script = <<-SHELL
  sudo apt update && apt upgrade -y
  sudo apt install mc -y
  sudo apt install sysstat -y
  sudo useradd -G sudo -m -p p@ssw0rd -s /bin/bash user
  sudo mkdir /home/user/.ssh
  sudo touch /home/user/.ssh/authorized_keys
  sudo mkdir /root/.ssh
  sudo touch /root/.ssh/authorized_keys
  sudo cat /vagrant/keys/key_public.pub > /home/user/.ssh/authorized_keys
  sudo cat /vagrant/keys/key_public.pub > /root/.ssh/authorized_keys
  sudo cat /vagrant/ansible/id_rsa_ansible.pub >> /root/.ssh/authorized_keys

  sudo echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
  sudo systemctl restart ssh.service
  sudo echo "alias ll='ls $LS_OPTIONS -l'" >> /root/.bashrc
  sudo echo "alias l='ls $LS_OPTIONS -lA'" >> /root/.bashrc
  sudo echo "alias ls='ls --color=auto'" >> /root/.bashrc
SHELL

######################################################################

Vagrant.configure("2") do |config|
	config.vm.box_check_update = false
  config.disksize.size = "50GB"

  ####################################################################
  (1..3).each do |i|
    config.vm.define "node0#{i}" do |node|
      node.vm.box = "#{BOX_DEBIAN}"
      node.vm.hostname = "node0#{i}"
      node.vm.network "private_network",ip: "10.0.0.#{110+i}"
      node.vm.provision "shell", inline: $script

      node.vm.provider "virtualbox" do |vb|
        vb.name = "node0#{i}"
        vb.memory = 2048
        vb.cpus = 1
      end
    end
  end

  ####################################################################
  config.vm.define "ansible" do |subconfig|
    subconfig.vm.box = "#{BOX_DEBIAN}"
    subconfig.vm.hostname = "ansible"
    subconfig.vm.network "private_network", ip: "10.0.0.100"

    subconfig.vm.provision "shell", inline: $script, run: "once"
    subconfig.vm.provision "shell", inline: $ansible, run: "once"
    subconfig.vm.provision "Copying configuration files to VM", type: "file", run: "once" do |vm|
      vm.source = Dir.pwd + "/ansible/ansible.cfg"
      vm.destination = "/tmp/ansible.cfg"
    end
    subconfig.vm.provision "shell", inline: "mv /tmp/ansible.cfg /root/ansible.cfg"


    subconfig.vm.provider "virtualbox" do |vm|
      vm.name = "ansible"
      vm.memory = 4096
      vm.cpus = 2
    end
  end

  ###############################################################
  config.vm.define "docker" do |subconfig|
    subconfig.vm.box = "#{BOX_DEBIAN}"
    subconfig.vm.hostname = "docker"
    subconfig.vm.network "private_network", ip: "10.0.0.104"
    subconfig.vm.provision "shell", inline: $script

    subconfig.vm.provider "virtualbox" do |vb|
      vb.name = "docker"
      vb.memory = 8192
      vb.cpus = 2
    end
  end

  ########################################################################
  config.vm.define "ubuntu" do |subconfig|
    subconfig.vm.box = "peru/ubuntu-20.04-desktop-amd64"
    subconfig.vm.box_version = "20210701.01"
    subconfig.vm.hostname = "ubuntu"
    subconfig.vm.network "private_network", ip: "10.0.0.105"
    subconfig.vm.provision "shell", inline: $script
    
    subconfig.vm.provider "virtualbox" do |vb|
      vb.name = "ubuntu"
      vb.memory = 4096
      vb.cpus = 2
    end
  end

  ###############################################################
  config.vm.define "gitlab" do |subconfig|
    subconfig.vm.box = "#{BOX_DEBIAN}"
    subconfig.vm.hostname = "gitlab"
    subconfig.vm.network "private_network", ip: "10.0.0.106"
    subconfig.vm.provision "shell", inline: $script

    subconfig.vm.provider "virtualbox" do |vb|
      vb.name = "gitlab"
      vb.memory = 8192
      vb.cpus = 2
    end
  end
end