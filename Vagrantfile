# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|

 
  config.vm.provision "shell", inline: <<-SHELL
  #sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config 
  #sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
  #sudo service ssh restart
  sudo apt-get update -y
  sudo apt-get install python -y
SHELL

  config.vm.define :web01 do |web01|
    web01.vm.box = "ubuntu/xenial64"
    web01.vm.hostname = "web01"
    web01.vm.network :private_network, ip: "172.20.0.21"
  end
  config.vm.define :web02 do |web02|
    web02.vm.box = "ubuntu/xenial64"
    web02.vm.hostname = "web02"
    web02.vm.network :private_network, ip: "172.20.0.22"
  end
  config.vm.define :web03 do |web03|
    web03.vm.box = "ubuntu/xenial64"
    web03.vm.hostname = "web03-avalanche"
    web03.vm.network :private_network, ip: "172.20.0.23"
  end
  config.vm.define :haproxy do |haproxy|
    haproxy.vm.box = "ubuntu/xenial64"
    haproxy.vm.hostname = "haproxy"
    haproxy.vm.network :private_network, ip: "172.20.0.24"
  end
  config.vm.box_check_update = false

  config.vm.provider "virtualbox" do |v|
    v.cpus = 1
    v.memory = "1024"
  end


  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "main.yml"
    ansible.inventory_path = "inventories/production"
  end
  
end
