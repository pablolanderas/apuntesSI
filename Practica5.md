# Practica 5

#### Índice

- [Interfaces](#interfaces)
- [Configuracíon de las interfaces](#conf_interfaces)
- [ARP](#arp)
- [Tablas de rutas](#tablas_de_rutas)
- [Resolucion de nombres](#resol_nombres)
- [Archivo de configuracion de DNS](#arch_conf_dns)
- [Utilizar el ordenador como router (ruteador)](#routeador)
- [Comandos utiles](#comandos_utiles)

## Interfaces <a id="interfaces">

Las interfaces se encuentran en ``/etc/network/`` y se configuran desde el archivo ``interfaces`` que esta en ese directorio

Estos son algunos comandos basicos de las interfaces:

- ``ifconfig``: muestra la configuración de las interfaces de red encendidas del sistema. Si quieres ver todas, las encendidas y las apagadas añadri ``-a`` al final

- ``ifup <interfaz>``: sirve para activar la interfaz de red especificada aplicandole las configuraciones

- ``ifdown <interfaz>``: sirve para descativar la interfaz de red especificada

- ``ip link show``: información sobre las interfaces de red en el sistema. Es similar a ``ifconfig``, pero es más moderno y ofrece capacidades extendidas

- ``ip address add <IP>/<mascara> dev <interfaz>``: sirve para cambiarle la ip y la mascara a una interfaz

### Configuracíon de las interfaces <a id="conf_interfaces">

Las interfaces se configuran desde el archivo ``/etc/network/interfaces``. Aqui encontraras información sobre el archivo

- La interfaz Loopback que sirve para la comunicación de aplicaciones de red alojadas en el mismo sistema, se identifica como ``lo`` y tiene configuración especial

- La linea ``auto <interface>``: indica que la interfaz seleccioanda debe activarse automaticamente cuando se arranque el sistema

- La linea ``iface <interface> <ip_addressing> <method>``: esta linea define la configuración de una interfaz de red especifica, recibe:
    
    - ``<interface>``: la interfaz que se quiere configurar

    - ``<ip_addresing>``: el tipo de direccionamiento que se utilizara (``inet`` para IPv4 y ``inet6`` para IPv6)

    - ``<metod>``: El método de configuración de la dirección IP para la interfaz, que puede ser ``dhcp`` para configuración automática (selección de IP dinámica) o ``static`` para configuración manual (estática).

        En las siguientes lineas se puede añadir informacion sobre la red en caso de ser una red estatica, se hace con una tabulacion, el parametro, un espacio y el valor. Estos son los posibles parametros:

        - *address*: la direccion IP

        - *netmask*: la mascara de red

        - *gateway*

        - *dns-nameservers*: la direccion IP del servidor DNS que desee utilizar

        - *broadcast*

        - *network*: la direccion de la red (la que la identifica, la primera direccion)

### ARP <a id="arp">

ARP es un protocolo de comunicaciones de la capa de enlace de datos, ​este es responsable de encontrar la dirección MAC del hardware que corresponde a una determinada dirección IP

Toda esta informacion se alamacena en tablas, y hay varios comandos con los que puedes ver y editar esta información:

- **Mostrar la tabla:** puedes mostrar la tabla con el comando ``arp -a``

- **Agregar una nueva entrada:** puedes agregar una nueva entrada a la tabal con el comando ``arp -s <IP> <MAC>``

- **Eliminar una entrada de la tabla:** puedes eliminar una entrada de la tabla con el comando ``arp -d <IP>``

- **Cambiar la MAC de una interfaz:** puedes cambiar la MAC de una interfaz con el siguiente comando ``ifconfig <interfaz> hw ether <MAC>``

### Tablas de rutas <a id="tablas_de_rutas">

A traves de ARP solo se puede acceder a los hosts de la misma red. Para poder acceder al resto de host deben de fijarse rutas IP. Es más limitado que las rutas dinamicas ya que tienes que añadir las rutas a mano pero para redes pequeñas puede ser util. Esto se hace a traves de tablas con los siguientes valores:

- **Destination:** identifica la red de destino

- **Gateway:** como acceder al destino

- **Genmask:** la mascara de red

- **Iface:** interfacz de red por la que se accede a la red de destino

Puedes interactuar con esta tabla de las siguientes maneras:

- **Mostrar la tabla:** para mostrar la tabla usar el comando ``route -n``

- **Añadir una ruta:** para añadir una nueva ruta se usa el comando ``route add –net <IP> netmask <mascara> <interfaz>``

- **Eliminar una ruta:** para eliminar una ruta se usa el comando ``route del -net <red> netmask <máscara> gw <gateway>``

- **Fijar ruta predeterminada (gateway):** para fijar una ruta predeterminada se utiliza el comando ``route add default gw <IP> <interfaz>``

### Resolucion de nombres <a id="resol_nombres">

Sirve para transformar "nombres" en IPs, para hacer esto hay dos formas:

- **El archivo ``/etc/hosts``:** en este archivo se pueden introducir la IP y el nombre que se le quiere asignar

- **Domain name service (DNS):** un servidor dedicado para realizar las traducciones de los nombres. Para configurar toda la información sobre el DNS a usar se hace en el archivo [``/etc/resolv.conf``](#arch_conf_dns). Hay que tener en cuenta que si se esta utilizando DHCP (selección de IP dinámica) este servidor lo asignara el protocolo por lo que te pisara los servidores que hayas configurado en el archivo

### Archivo de configuracion de DNS <a id="arch_conf_dns">

En el archivo ``/etc/resolv.conf`` se puede configurar el uso del servidor de nombes con los siguientes parametros:

- ``search``: nos permite definir una lista de dominios que se utilizarán para completar los nombre cortos, antes de buscarlos. Los dominios de la lista deberán de ir separados por espacios o tabuladores, no pudiendo superar los 6 dominios, con un total de 256 caracteres. Un ejemplo seria por ejemplo si estubiera el siguiente codigo ``search zeppelinux.es zeppelinux.com``, si se itentara hacer un ``ping www``, lo primero que hará será buscar *www.zeppelinux.es*, si no se encuentra, probará con *www.zeppelinux.com* y por último intentará resolver el nombre *www*

- ``nameserver``: se introduce la direccion IP de servidor DNS, se pueden introducir varios, si falla el primero prueba con el siguiente en orden descendente.

### Utilizar el ordenador como router (ruteador) <a id="routeador">

Esto se puede activar para utilizar el ordenador como gateway. Para hacer esto permamente en el archivo ``/etc/sysctl.conf`` hay que decomentar o poner a 1 la linea ``net.ipv4.ip_forward = 1``. 
Una vez hecho esto cuando arranques funcionara como routeador, pero para cambiar el kernel y que no haya que reiniciarlo, puedes hacer ``sysctl -w net.ipv4.ip_forward=1``, este comando lo pone a 1, pero cuando se reinicia se queda como esta en el fichero.

### Comandos utiles <a id="comandos_utiles">

- ``netstat``: te da informacion sobre la red, las conexiones activas, estadisticas de interfaz de red, tablas de enrutamiento, ...

- ``traceroute``: te indica la ruta de un paquete hasta su destino 

- ``apt-get update``: utilizarlo para comprobar si funciona adecuadamente la red