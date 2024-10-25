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

Para realizar esta actividad he tenido que modificar varios archivos. He comenzado creando en el Vagrantfile un script con provisiones para que lo requerido en este ejercicio se cree de manera automática; para ello he añadido los cambios que deben darse en archivos concretos.

Una vez creada la máquina Tierra con la instalación de bind9:

/*--------------------------------------------TIERRA---------------------------------------*/

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

// Definiciond e la ACL
acl recursivas {
	127.0.0.0/8;
	192.168.57.0/24;
};


options {
directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	forwarders {
	    208.67.222.222;
	};
	
	forward only;
	
	allow-transfer {none; };	
	listen-on port 53 { 127.0.0.1; 192.168.57.103; };
	
	recursion yes;
	allow-recursion {recursivas; };
	
	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation yes;

	//listen-on-v6 { any; };
};

```
3. he creado dos archivos en la ruta /etc/lib/bind, los cuales han sido:

```bash

/etc/lib/bind/tierra.sistema.tets

$TTL	86400
@ IN SOA debian.tierra.sistema.test. admin.tierra.sistema.test. (
       202410231   ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        7200     ; Negative Cache TTL
)

@	 IN NS	 debian.tierra.sistema.test.
debian.tierra.sistema.test. IN A	 192.168.57.103
venus.sistema.test. IN A       192.168.57.102
marte.sistema.test IN A       192.168.57.104
mercurio.sistema.test IN A       192.168.57.101

; Alias
ns1 IN CNAME tierra.sistema.test.
ns2 IN CNAME venus.sistema.test.
mail IN CNAME marte.sistema.test.
ns3 IN CNAME mercurio.sistema.test.

;Registro MR
@ IN MX 10 marte.sistema.test

```
y 

```bash

etc/lib/bind/tierra.sistema.test.rev


$TTL    86400
@  IN SOA debian.tierra.sistema.test. admin.tierra.sistema.test. (
       202410231   ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        7200      ; Negative Cache TTL
)
@        IN NS   debian.tierra.sistema.test.
; Registros PTR
103	  IN PTR      debian.tierra.sistema.test.
102      IN PTR      venus.sistema.test.
104      IN PTR      marte.tierra.sistema.test.
101      IN PTR      mercurio.sistema.test.

```

/*--------------------------------------------VENUS----------------------------------------*/


En venus:

1. He modificado dos archivos en la ruta /etc/bind, los cuales han sido


```bash

/etc/bind/named.conf.options

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
  

 ```bash

 /etc/bind/named.conf.local


zone "venus.sistema.test" {
            type master;
            file "/var/lib/bind/venus.sistema.test";
            masters { 192.168.57.103cd ; };  // IP de tierra
          };

```
2. He creado en /var/lib/bind/venus.sistema.test

```bash
   $TTL    86400
  @ IN SOA debian.venus.sistema.test. admin.venus.sistema.test. (
       202410231   ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        7200       ; Negative Cache TTL
  )
  @        IN NS   debian.venus.sistema.test.
  debian.venus.sistema.test. IN A         192.168.57.102
  ns1 IN CNAME tierra.sistema.test.
  ns2 IN CNAME venus.sistema.test. 

```
Todos los archivos modificados anteriormente han sido copiados en /vagrant, en carpetas separadas para que no hubiera solapamiento entre los archivos de Venus y de Tierra. De este modo, en las provisiones que indico en el Vagrantfile, cuando creemos las máquinas desde cero, estas cogerán los archivos que se han copiado anteriormente en /vagrant y los sustituirán/añadirán en las nuevas máquinas, teniendo así los ajustes que necesitamos.

/*--------------------------------------------MARTE----------------------------------------*/

Para Marte, aunque solo había que hacer algunas modificaciones en tierra.sistema.test y venus.sistema.test, yo he creado una provisión para crear otra máquina llamada Marte, la cual actúa como servidor de correo.

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
## Notas

Como se puede observar en la documentación, hay un .txt llamado VagrantfileV0, en este archivo se encuentra el primer Vagrantfile que creé, en el cual añadí los cambios que hacía en cada srchivo en bash, en vez de copiar directamente el archivo ya modificado. Como temo que algún comando pueda tener error, he preferido quitarlo y usar las provisiones justas de la creación de la máquina más la parte donde le decimos que añada dichos archivos de /vagrant/tierra o /vagrant/venus. Como esa parte me llevó bastante tiempo redactarla, no he querido eliminar del todo el, sino dejarla ahí como información extra.
