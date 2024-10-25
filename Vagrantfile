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
  tierra.vm.provision "shell", inline: <<-SHELL
    # Actualizar paquetes
    sudo apt-get update
    sudo apt-get upgrade
     # Instalar bind9 si no está instalado
    sudo apt-get install -y bind9 dnsutils
    # Modificar el archivo /etc/default/named
    sudo echo 'OPTIONS="-u bind -4"' | sudo tee /etc/default/named
    # Reiniciar el servicio bind9
    sudo sudo systemctl restart bind9
  SHELL
  # provisonar sólo este bloque 'vagrant provision tierra --provision-with config'
  tierra.vm.provision "shell", inline: <<-SHELL
    cp /vagrant/tierra/named /etc/default
    cp /vagrant/tierra/named.conf.* /etc/bind
    cp /vagrant/tierra/tierra.sistema.test /var/lib/bind
    cp /vagrant/tierra/tierra.sistema.test.rev /var/lib/bind
    sudo systemctl restart named
    sudo systemctl restart bind9
  SHELL
end #tierra
  # Servidor ESCLAVO Venus DNS
  config.vm.define "venus" do |venus|
    venus.vm.hostname = "venus.sistema.test"
    venus.vm.network "private_network", ip: "192.168.57.102"
    venus.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get upgrade
      sudo apt-get install -y bind9 dnsutils
       # Reiniciar el servicio bind9
    sudo systemctl restart bind9
    SHELL
      # provisonar sólo este bloque 'vagrant provision venus --provision-with config'
      venus.vm.provision "shell", inline: <<-SHELL
        cp /vagrant/venus/named /etc/default
        cp /vagrant/venus/named.conf.* /etc/bind
        cp /vagrant/venus/venus.tierra.test /var/lib/bind
        sudo systemctl restart named
        sudo systemctl restart bind9
        
    SHELL
  end #venus
  
  # Servidor Marte (Correo)
  config.vm.define "marte" do |marte|
    marte.vm.hostname = "marte.sistema.test"
    marte.vm.network "private_network", ip: "192.168.57.104"
    marte.vm.provision "shell", inline: <<-SHELL
      # Actualizar paquetes
      sudo apt-get update
      sudo apt-get upgrade
      # Instalar Postfix
      sudo apt-get install -y postfix
      # Configurar Postfix
      postconf -e "myhostname = marte.sistema.test"
      postconf -e "mydomain = sistema.test"
      postconf -e "myorigin = /etc/mailname"
      postconf -e "inet_interfaces = all"
      postconf -e "inet_protocols = all"
      # Reiniciar Postfix
      systemctl restart postfix
    SHELL
    # Agregar registros DNS
    marte.vm.provision "shell", inline: <<-SHELL
      echo "@ IN MX 10 marte.sistema.test." >> /etc/bind/tierra.sistema.test
      echo "mail IN CNAME marte.sistema.test." >> /etc/bind/tierra.sistema.test
      # Reiniciar BIND para aplicar los cambios
      systemctl restart bind9
    SHELL
  end #marte

end

    