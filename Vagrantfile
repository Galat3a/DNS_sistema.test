# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  # config.vbguest.auto_update = false
  config.ssh.insert_key = false

  # Servidor Venus
  config.vm.define "venus" do |venus|
    master.vm.hostname = "venus.sistema.test"
    master.vm.network "private_network", ip: "192.168.57.102"
    master.vm.provision "shell", name: "update", inline: <<-SHELL
      apt-get update
      apt-get install -y bind9 dnsutils
    SHELL
    # provisonar sólo este bloque   'vagrant provision venus --provision-with config'
    venus.vm.provision "shell", name: "config", inline: <<-SHELL
      cp /vagrant/named /etc/default
      cp /vagrant/named.conf.* /etc/bind
      cp /vagrant/deaw.test.dns /var/lib/bind
      cp /vagrant/192.168.57.dns /var/lib/bind
      systemctl restart named
    SHELL
  end #venus

    # Servidor Tierra
    config.vm.define "tierra" do |tierra|
      master.vm.hostname = "tierra.sistema.test"
      master.vm.network "private_network", ip: "192.168.57.103"
      master.vm.provision "shell", name: "update", inline: <<-SHELL
        apt-get update
        apt-get install -y bind9 dnsutils
      SHELL
      # provisonar sólo este bloque 'vagrant provision tierra --provision-with config'
      tierra.vm.provision "shell", name: "config", inline: <<-SHELL
        cp /vagrant/named /etc/default
        cp /vagrant/named.conf.* /etc/bind
        cp /vagrant/deaw.test.dns /var/lib/bind
        cp /vagrant/192.168.57.dns /var/lib/bind
        systemctl restart named
      SHELL
  end #tierra