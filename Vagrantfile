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
      allow-recursion { 192.168.57.0/24; };
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
    listen-on port 53 { 127.0.0.1; 192.168.57.103; };
    
    recursion yes;
    allow-recursion { recursivas; };

    dnssec-validation yes;

    forwarders {
        208.67.222.222;  // OpenDNS
    };

    forward only;  // Esto asegura que solo se reenvíen consultas

    // listen-on-v6 { any; };
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
        7200       ; Negative Cache TTL
    ;
    @        IN NS   debian.tierra.sistema.test.
    debian.tierra.sistema.test. IN A         192.168.57.103
    ; Alias
    ns1 IN CNAME tierra.sistema.test.
    ns2 IN CNAME venus.sistema.test.
    ns3 IN CNAME 192.168.57.101
    ;Registro MR
    @ IN MX 10 marte.sistema.test 
EOF
    # Modificar el archivo /etc/bind/named.conf.local para la zona inversa y para añadir a venus como esclavo

    sudo tee -a /etc/bind/named.conf.local << EOF
    zone "tierra.sistema.test" {
      type master;
      file "/etc/bind/tierra.sistema.test";
      allow-transfer { 192.168.57.102; };  // IP de venus
    };
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
        7200       ; Negative Cache TTL
    ;
    @        IN NS   debian.tierra.sistema.test.
    103             IN PTR      debian.tierra.sistema.test.
EOF
  SHELL
  # provisonar sólo este bloque 'vagrant provision tierra --provision-with config'
  tierra.vm.provision "shell", inline: <<-SHELL
    cp /vagrant/tierra/named /etc/default
    cp /vagrant/tierra/named.conf.* /etc/bind
    cp /vagrant/tierra/tierra.sistema.test /var/lib/bind
    cp /vagrant/tierra/tierra.sistema.test.rev /var/lib/bind
    systemctl restart named
    systemctl restart bind9
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
            masters { 192.168.57.103cd ; };  // IP de tierra
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
        7200       ; Negative Cache TTL
      ;
      @        IN NS   debian.venus.sistema.test.
      debian.venus.sistema.test. IN A         192.168.57.102
      ns1 IN CNAME tierra.sistema.test.
      ns2 IN CNAME venus.sistema.test. 
EOF
    SHELL
      # provisonar sólo este bloque 'vagrant provision venus --provision-with config'
      venus.vm.provision "shell", inline: <<-SHELL
        cp /vagrant/venus/named /etc/default
        cp /vagrant/venus/named.conf.* /etc/bind
        cp /vagrant/venus/venus.tierra.test /var/lib/bind
        systemctl restart named
        systemctl restart bind9
    SHELL
  end #venus
  
  # Servidor Marte (Correo)
  config.vm.define "marte" do |marte|
    marte.vm.hostname = "marte.sistema.test"
    marte.vm.network "private_network", ip: "192.168.57.104"
    marte.vm.provision "shell", inline: <<-SHELL
      # Actualizar paquetes
      apt-get update
      # Instalar Postfix
      apt-get install -y postfix
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

    