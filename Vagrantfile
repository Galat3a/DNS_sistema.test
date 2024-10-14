# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  # config.vbguest.auto_update = false
  config.ssh.insert_key = false

 # Servidor MASTER Tierra
 config.vm.define "tierra" do |tierra|
  tierra.vm.hostname = "tierra.sistema.test"
  tierra.vm.network "private_network", ip: "192.168.57.103"
  tierra.vm.provision "shell", name: "update", inline: <<-SHELL
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

  # Servidor ESCLAVO Venus DNS
  config.vm.define "venus" do |venus|
    venus.vm.hostname = "venus.sistema.test"
    venus.vm.network "private_network", ip: "192.168.57.102"
    venus.vm.provision "shell", name: "update", inline: <<-SHELL
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

end

    