# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  # config.vbguest.auto_update = false
  config.ssh.insert_key = false

  # Servidor DNS
  config.vm.define "master" do |master|
    master.vm.hostname = "master.deaw.test"
    master.vm.network "private_network", ip: "192.168.57.10"
    master.vm.provision "shell", name: "update", inline: <<-SHELL
      apt-get update
      apt-get install -y bind9 dnsutils
    SHELL
    #provisonar sólo este bloque 'vagrant provision master --provision-with config'
    master.vm.provision "shell", name: "config", inline: <<-SHELL
      cp /vagrant/named /etc/default
      cp /vagrant/named.conf.* /etc/bind
      cp /vagrant/deaw.test.dns /var/lib/bind
      cp /vagrant/192.168.57.dns /var/lib/bind
      systemctl restart named
    SHELL
  end #master