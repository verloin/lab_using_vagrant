
# -*- mode: ruby -*-
# vi: set ft=ruby :
# vagrant plugin install vagrant-vbguest
# vagrant plugin install vagrant-disksize

BOX_DEBIAN = "debian/buster64"
# BOX_DEBIAN_DESKTOP = "boxomatic/debian-10-mate"
BOX_UBUNTU_DESKTOP = "peru/ubuntu-20.04-desktop-amd64"
# BOX_VERSION_UBUNTU_DESKTOP = "20211016.01"

# $hosts
#       10.0.0.100 ansible
#       10.0.0.101 zabbix
#       10.0.0.102 elk
#       10.0.0.104 docker
#       10.0.0.105 
#       10.0.0.106 gitlab
#       10.0.0.110 ubuntu
#       10.0.0.111 node01
#       10.0.0.112 node02
#       10.0.0.113 node03


$base_script = <<-SHELL
  sudo apt update && apt upgrade -y
  sudo apt install mc -y
  sudo apt install sysstat -y
  sudo mkdir /root/.ssh
  sudo touch /root/.ssh/authorized_keys

  sudo cat /vagrant/keys/key_public.pub > /root/.ssh/authorized_keys
  sudo echo "\n" >> /root/.ssh/authorized_keys
  sudo cat /vagrant/ansible/id_rsa_ansible.pub >> /root/.ssh/authorized_keys
  sudo echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
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
      node.vm.provision "shell", inline: $base_script, run: "once"

      node.vm.provider "virtualbox" do |vb|
        vb.name = "node0#{i}"
        vb.memory = 2048
        vb.cpus = 1
      end
    end
  end

  ########################################################################
  config.vm.define "zabbix" do |sub|
    sub.vm.box = "#{BOX_DEBIAN}"
    sub.vm.hostname = "zabbix"
    sub.vm.network "private_network", ip: "10.0.0.101"

    sub.vm.provision "shell", inline: $base_script, run: "once"
  
    sub.vm.provider "virtualbox" do |vb|
      vb.name = "zabbix"
      vb.memory = 8192
      vb.cpus = 2
    end
  end

  ########################################################################
  config.vm.define "elk" do |sub|
    sub.vm.box = "#{BOX_DEBIAN}"
    sub.vm.hostname = "elk"
    sub.vm.network "private_network", ip: "10.0.0.102"

    sub.vm.provision "shell", inline: $base_script, run: "once"
  
    sub.vm.provider "virtualbox" do |vb|
      vb.name = "elk"
      vb.memory = 4096
      vb.cpus = 2
    end
  end

  ###############################################################
  config.vm.define "docker" do |sub|
    sub.vm.box = "#{BOX_DEBIAN}"
    sub.vm.hostname = "docker"
    sub.vm.network "private_network", ip: "10.0.0.104"
    sub.vm.provision "shell", inline: $base_script, run: "once"

    sub.vm.provider "virtualbox" do |vb|
      vb.name = "docker"
      vb.memory = 8192
      vb.cpus = 2
    end
  end

  ########################################################################
  config.vm.define "ubuntu" do |sub|
    sub.vm.box = "#{BOX_UBUNTU_DESKTOP}"
    sub.vm.hostname = "ubuntu"
    sub.vm.network "private_network", ip: "10.0.0.110"

    sub.vm.provision "Copying folder with configuration files to VM", type: "file", run: "once" do |vb|
      vb.source = Dir.pwd + "/"
      vb.destination = "/tmp/ubuntu"
    end

    sub.vm.provision "shell", inline: $ubuntu_base, run: "once"
    sub.vm.provision "shell", inline: $ubuntu_ansible, run: "once"
  
    sub.vm.provider "virtualbox" do |vb|
      vb.name = "ubuntu"
      vb.memory = 8192
      vb.cpus = 2
    end
  end

  ###############################################################
  config.vm.define "gitlab" do |sub|
    sub.vm.box = "#{BOX_DEBIAN}"
    sub.vm.hostname = "gitlab"
    sub.vm.network "private_network", ip: "10.0.0.106"
    sub.vm.provision "shell", inline: $base_script, run: "once"

    sub.vm.provider "virtualbox" do |vb|
      vb.name = "gitlab"
      vb.memory = 8192
      vb.cpus = 2
    end
  end

  ####################################################################
  # config.vm.define "ansible" do |subconfig|
  #   subconfig.vm.box = "#{BOX_DEBIAN}"
  #   subconfig.vm.hostname = "ansible"
  #   subconfig.vm.network "private_network", ip: "10.0.0.100"

  #   subconfig.vm.provision "shell", inline: $script, run: "once"
  #   subconfig.vm.provision "shell", inline: $ansible, run: "once"

  #   subconfig.vm.provider "virtualbox" do |vm|
  #     vm.name = "ansible"
  #     vm.memory = 4096
  #     vm.cpus = 2
  #   end
  # end
end

##################################################################
####    SHELL  ####
##################################################################


$ubuntu_base = <<-SHELL
  sudo apt update && apt upgrade -y
  sudo apt install mc -y
  sudo apt install sysstat -y
  sudo apt-get install virtualbox-guest-x11
  sudo apt install snapd
  sudo snap install pycharm-community --classic

  sudo useradd -G sudo -m -p p@ssw0rd -s /bin/bash yury
  sudo mkdir /home/yury/.ssh /root/.ssh
  
  sudo cp -r /tmp/ubuntu/keys/key_public.pub /root/.ssh/authorized_keys
  sudo cp -r /tmp/ubuntu/keys/key_public.pub /home/yury/.ssh/authorized_keys

  sudo echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
  sudo systemctl restart ssh.service
  sudo echo "alias ll='ls $LS_OPTIONS -l'" >> /root/.bashrc
  sudo echo "alias l='ls $LS_OPTIONS -lA'" >> /root/.bashrc
  sudo echo "alias ls='ls --color=auto'" >> /root/.bashrc
SHELL

$ubuntu_ansible = <<-SHELL
  sudo apt install software-properties-common
  sudo apt-add-repository ppa:ansible/ansible
  sudo apt update
  sudo apt install ansible -y
  sudo apt install git python3-pip -y

  mkdir /home/yury/ansible /home/yury/ansible/group_vars /home/yury/ansible/playbooks /home/yury/ansible/roles

  sudo cp -r /tmp/ubuntu/ansible/roles /home/yury/ansible/
  sudo cp -r /tmp/ubuntu/ansible/playbooks /home/yury/ansible/
  sudo cp -r /tmp/ubuntu/ansible/hosts /home/yury/ansible/
  sudo cp -r /tmp/ubuntu/ansible/ansible.cfg  /home/yury/
  sudo cp -r /tmp/ubuntu/ansible/id_rsa_ansible.pub /home/yury/.ssh/
  sudo cp -r /tmp/ubuntu/ansible/id_rsa_ansible /home/yury/.ssh/
SHELL

#  subconfig.vm.network "private_network", ip: "10.0.0.101"
#  subconfig.vm.network "forwarded_port", id: "ssh", host: 2101, guest: 22
#  subconfig.vm.network "forwarded_port", guest:80, host:8080
#  subconfig.vm.network "forwarded_port", guest:443, host:8443
   
# subconfig.vm.synced_folder "ansible", "/root/ansible", create: true, owner: "root", group: "root"

# subconfig.vm.provision "Copying configuration files to VM", type: "file", run: "once" do |vm|
#   vm.source = Dir.pwd + "/ansible/ansible.cfg"
#   vm.destination = "/tmp/ansible.cfg"
# end