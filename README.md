# DNS_sistema.test

Este repositorio contiene configuraciones y scripts relacionados con el sistema DNS.

## Licencia

Puedes elegir una licencia para tu proyecto en el siguiente enlace: [Choose a License](https://choosealicense.com/licenses)

## Plantilla de Commits

Añade una plantilla para tus commits utilizando el siguiente comando:

```bash
git config --global commit.template "C:\Users\DAW-A.DESKTOP-ECOE3HP\.gittemplate"
```
# Resumen tarea

Para realizar esta actividad he tenido que modificar varios archivos. He comenzado creando en vagrantfile un script con provisiones para que lo requerido en este ejercicio se cree de manera automatica, para ello he añadido los cambios que deben de darse en archivos concretos.

Una vez creada la maquina tierra con la instalación de bind9:

1. He modificado el archivo named de la siguiente ruta:

```bash
/etc/default/named

#
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-u bind -4"

```

2. He modicicado dos archivos, los cuales son: 

/ect/bind/named.conf.local

```bash
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "tierra.sistema.test" {
    type master;
    file "/etc/bind/tierra.sistema.test";
    allow-transfer { 192.168.57.102; };  // IP de venus
};

zone "57.168.192.in-addr.arpa" {
    type master;
    file "/var/lib/bind/tierra.sistema.test.rev";
};
```

 y

/etc/bind/named.conf.options

```bash
;
; tierra.sistema.test
;
$TTL	86400
@ IN SOA debian.tierra.sistema.test. admin.tierra.sistema.test. (
       202410231   ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        7200 )     ; Negative Cache TTL (2 horas)
;
@	 IN NS	 debian.tierra.sistema.test.
debian.tierra.sistema.test. IN A	 192.168.57.103

; Alias
ns1 IN CNAME tierra.sistema.test.
ns2 IN CNAME venus.sistema.test.
mail IN CNAME marte.sistema.test.

;Registro MR
@ IN MX 10 marte.sistema.test

y var/lib/bind/tierra.sistema.rev
; 
; 57.168.192
;
$TTL    86400
@  IN SOA debian.tierra.sistema.test. admin.tierra.sistema.test. (
       202410231   ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        7200 )     ; Negative Cache TTL (2 horas)
;
@        IN NS   debian.tierra.sistema.test.
103		IN PTR      debian.tierra.sistema.test
```
En venus:

1. He modificado dos archivos en la ruta /etc/bind, los cuales han sido

/etc/bind/named.conf.options

```bash
acl recursivas {
    127.0.0.0/8;
    192.168.57.0/24;
  };

  options {
    directory "/var/cache/bind";

    allow-transfer { none; };
    listen-on port 53 { 127.0.0.1; 192.168.57.102; };
    
    recursion yes;
    allow-recursion { recursivas; };

    dnssec-validation yes;

    forwarders {
        208.67.222.222;  // OpenDNS
    };

    forward only;  // Esto asegura que solo se reenvíen consultas

    // listen-on-v6 { any; };
  };

  ```
  /etc/bind/named.conf.local

 ```bash

//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "venus.sistema.test" {
            type master;
            file "/var/lib/bind/venus.sistema.test";
            masters { 192.168.57.103cd ; };  // IP de tierra
          };

```
2. He creado en /var/lib/bind/venus.sistema.test

```bash
  ;
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

```
Todos los archivos mencionados anteriormente los he copiado en Vagrant, cada uno en una carpeta propia para que no se pisen aquellos que se llaman igual, estos se sustituiran por los originales en la creacion de las maquinas cuando ejecutemos estas con vagrant up a traves de las provisiones indicadas en vagrantfile, sobre crear esta copias en vagrantfile me di cuenta tarde, por lo que en mi provisionamiento también tengo incluido las modificaciones de cada archivo por separado en desglosado en comandos bash. 

Para marte, aunque solo habia que hacer algunas modificaciones en tierra.sistema.test y venus.sistema.tes, yo he creado una provision para crear otra maquina llamada marte la cual actua como servidor de correo. 

```bash
  
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
```