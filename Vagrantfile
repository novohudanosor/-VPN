  GNU nano 6.2                                                                              Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.vm.box = "ubuntu/jammy64"
  config.vm.provider "virtualbox" do |box|
      box.memory = 512
      box.cpus = 1
  end

  config.vm.define "server" do |server|
      server.vm.hostname = "server.loc"
      server.vm.network "private_network", ip: "192.168.56.10"
  end

  config.vm.define "client" do |client|
      client.vm.hostname = "client.loc"
      client.vm.network "private_network", ip: "192.168.56.20"
  end

end
