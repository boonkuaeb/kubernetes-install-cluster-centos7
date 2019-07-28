# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"
  config.vm.provision "shell", path: "provision/node.sh", privileged: true

  # Master
  (1..1).each do |number|
    config.vm.define "master#{number}" do |node|
      node.vm.network "private_network", ip: "192.168.99.10#{number}"
      node.vm.hostname = "master#{number}"
    end

  end

  # Node
  (1..2).each do |number|
    config.vm.define "node#{number}" do |node|
      node.vm.network "private_network", ip: "192.168.99.11#{number}"
      node.vm.hostname = "node#{number}"
    end
  end


  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 1
  end
end
