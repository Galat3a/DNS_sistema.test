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
    apt-get update
     # Instalar bind9 si no está instalado
    apt-get install -y bind9 dnsutils
    # Modificar el archivo /etc/default/named
    echo 'OPTIONS="-u bind -4"' | sudo tee /etc/default/named
    # Reiniciar el servicio bind9
    sudo systemctl restart bind9
    # Modificar el archivo /etc/bind/named.conf.options
    # Modificar el archivo /etc/bind/named.conf.options
    sudo tee /etc/bind/named.conf.options << EOF
      allow-transfer { none; };
      listen-on port 53 { 192.168.57.0; };
      recursion yes;
      allow-recursion { 192.168.57.0/24; };  # Asegúrate de que esto sea correcto
      dnssec-validation yes;
EOF
  # Reiniciar el servicio bind9
  sudo systemctl restart bind9
  # Modificar el archivo /etc/bind/named.conf.options
  sudo tee /etc/bind/named.conf.options << EOF
    acl recursivas {
      127.0.0.0/8;
      192.168.57.0/24;
      };

    options {
      directory "/var/cache/bind";

      allow-transfer { none; };
      listen-on port 53 { 192.168.57.0; };

      recursion yes;
      allow-recursion { recursivas; };

      dnssec-validation yes;

      // listen-on-v6 { any; };
    };
EOF  
  # Modificar el archivo /etc/bind/named.conf.local
  sudo tee /etc/bind/named.conf.local << EOF
zone "tierra.sistema.test" {
        type master;
        file "/var/lib/bind/tierra.sistema.test";
};
EOF
  #Creacion del archivo /var/lib/bind/tierra.sistema.test
  sudo tee /etc/bind/tierra.sistema.test << EOF
  ;
  ; tierra.sistema.test
  ;
  \$TTL    86400
  @ IN SOA debian.tierra.sistema.test. admin.tierra.sistema.test. (
       202410231   ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        86400 )    ; Negative Cache TTL
  ;
  @        IN NS   debian.tierra.sistema.test.
  debian.tierra.sistema.test. IN A         192.168.57.103
EOF
# Modificar el archivo /etc/bind/named.conf.local para la zona inversa y para añadir a venus como esclavo

sudo tee -a /etc/bind/named.conf.local << EOF
zone "tierra.sistema.test" {
    type master;
    file "/etc/bind/tierra.sistema.test";
    allow-transfer { 192.168.57.102; };  // IP de venus
};
EOF
EOF
# Crear el archivo /var/lib/bind/tierra.sistema.test.rev
sudo tee /var/lib/bind/tierra.sistema.test.rev << EOF
;
; 57.168.192
;
\$TTL    86400
@  IN SOA debian.tierra.sistema.test. admin.tierra.sistema.test. (
       202410231   ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        86400 )    ; Negative Cache TTL
;
@        IN NS   debian.tierra.sistema.test.
103             IN PTR      debian.tierra.sistema.test.
EOF
  SHELL
  # provisonar sólo este bloque 'vagrant provision tierra --provision-with config'
  tierra.vm.provision "shell", inline: <<-SHELL
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
    venus.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y bind9 dnsutils
      # Modificar el archivo /etc/bind/named.conf.local
      sudo tee /etc/bind/named.conf.local << EOF
          zone "venus.sistema.test" {
            type master;
            file "/var/lib/bind/venus.sistema.test";
          };
EOF
  #Creacion del archivo /var/lib/bind/venus.sistema.test
  sudo tee /etc/bind/venus.sistema.test << EOF
  ; venus.sistema.test
  ;
  \$TTL    86400
  @ IN SOA debian.venus.sistema.test. admin.venus.sistema.test. (
       202410231   ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        86400 )    ; Negative Cache TTL
  ;
  @        IN NS   debian.venus.sistema.test.
  debian.venus.sistema.test. IN A         192.168.57.102
EOF
    SHELL
    # provisonar sólo este bloque   'vagrant provision venus --provision-with config'
    venus.vm.provision "shell", inline: <<-SHELL
      cp /vagrant/named /etc/default
      cp /vagrant/named.conf.* /etc/bind
      cp /vagrant/deaw.test.dns /var/lib/bind
      cp /vagrant/192.168.57.dns /var/lib/bind
      systemctl restart named
    SHELL
  end #venus

end

    