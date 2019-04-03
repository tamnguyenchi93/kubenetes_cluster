# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box = "ubuntu/xenial64"
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "192.168.205.10"
    master.vm.network :forwarded_port, guest: 22, host: 32222, id: "ssh"
    master.vm.provider "virtualbox" do |master|
      master.memory = "2404"
      master.cpus = "2"
    end
    master.vm.provision 'shell', inline: <<-SHELL
      sed -i 's/PermitRootLogin.*$/PermitRootLogin yes/g' /etc/ssh/sshd_config
      sed -i 's/PasswordAuthentication.*$/PasswordAuthentication yes/g' /etc/ssh/sshd_config
      service ssh restart
      apt update
      apt install software-properties-common
      add-apt-repository ppa:deadsnakes/ppa
      apt update
      apt -y install python3.5
      echo -e 'nctam123\nnctam123' | passwd root
    SHELL
  end
	
  config.vm.define "worker1" do |worker1|
    worker1.vm.box = "ubuntu/xenial64"
    worker1.vm.hostname = "worker1"
    worker1.vm.network :private_network, ip: "192.168.205.11"
    worker1.vm.network :forwarded_port, guest: 22, host: 32221, id: "ssh"
    worker1.vm.provider "virtualbox" do |worker1|
      worker1.memory = "2404"
      worker1.cpus = "2"
    end
    worker1.vm.provision 'shell', inline: <<-SHELL
      #echo 'root:nctam1234' | chpasswd
      sed -i 's/PermitRootLogin.*$/PermitRootLogin yes/g' /etc/ssh/sshd_config
      sed -i 's/PasswordAuthentication.*$/PasswordAuthentication yes/g' /etc/ssh/sshd_config
      service ssh restart
      apt update
      apt install software-properties-common
      add-apt-repository ppa:deadsnakes/ppa
      apt update
      apt -y install python3.5
      echo -e 'nctam123\nnctam123' | passwd root
    SHELL
  end
  
  # config.vm.define "worker2" do |worker2|
  #   worker2.vm.box = "ubuntu/xenial64"
  #   worker2.vm.hostname = "worker2"
  #   worker2.vm.network :private_network, ip: "192.168.205.12"
  #   worker2.vm.network :forwarded_port, guest: 22, host: 32223, id: "ssh"
  #   worker2.vm.provider "virtualbox" do |worker2|
  #     worker2.memory = "2404"
  #     worker2.cpus = "2"
  #   end
  #   worker2.vm.provision 'shell', inline: <<-SHELL
  #     #echo 'root:nctam1234' | chpasswd
  #     sed -i 's/PermitRootLogin.*$/PermitRootLogin yes/g' /etc/ssh/sshd_config
  #     sed -i 's/PasswordAuthentication.*$/PasswordAuthentication yes/g' /etc/ssh/sshd_config
  #     service ssh restart
  #     apt update
  #     apt install software-properties-common
  #     add-apt-repository ppa:deadsnakes/ppa
  #     apt update
  #     apt -y install python3.5
  #     echo -e 'nctam123\nnctam123' | passwd root
  #   SHELL
  # end
#   config.vm.box = "ubuntu/xenial64"
end
