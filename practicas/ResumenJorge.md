# Sistemas Informáticos
Contenidos del examen parcial: ficheros, usuarios, software, redes.

- [Preguntas sueltas](#preguntas-sueltas)
- [Práctica 2: Disco duro, particiones, apt-get, fstab, copy -r](#práctica-2-disco-duro-particiones-apt-get-fstab-copy--r)
    - [Creación de disco](#creación-de-disco)
    - [Particiones y Filesystem](#particiones-y-filesystem)
    - [Apt-get install / cache](#apt-get-install--cache)
    - [Home en disco duro externo](#home-en-disco-duro-externo)
    - [Sustituir tres particiones por una que ocupe tamaño total](#sustituir-tres-particiones-por-una-que-ocupe-tamaño-total)
- [Práctica 3: Usuarios, grupos, GECOS, john, macros](#práctica-3-usuarios-grupos-gecos-john-macros)
    - [Archivos de usuarios, grupos y contraseñas](#archivos-de-usuarios-grupos-y-contraseñas)
    - [Creación de usuarios](#creación-de-usuarios)
    - [Cambiar a contraseña segura](#cambiar-a-contraseña-segura)
    - [John the Ripper](#john-the-ripper)
    - [Borrado de usuarios](#borrado-de-usuarios)
    - [Creación de scripts con AWK](#creación-de-scripts-con-awk)
- [Práctica 4: Software, descarga de archivos](#práctica-4-software-descarga-de-archivos)
    - [1ª forma de instalación software: Código fuente y compilado (archivos .tar)](#1ª-forma-de-instalación-software-código-fuente-y-compilado-archivos-tar)
    - [Repositorios y llaves (keys)](#repositorios-y-llaves-keys)
## Preguntas Sueltas
- En que inodo esta x archivo o directorio?
    ```bash
    ls -ia #saca el inodo de todo 
        # el numero que acompaña al . 
        # es el de la propia carpeta.
    ```
- Cuanto bloques ocupa cada inode?
    ```bash
    En ls -l, el numero que sigue a los permisos.
    ```    
- Para copiar un archivo random de tamaño 512MB 
    ```bash
    dd if=/dev/random of=/root/random bs=512MB count=1
    # Útil para rellenar carpetas de x tamaño
    ```
- Calculos medianamente complejos en scripts
    Si queremos sumar todas las cifras de un numero en un script usando `sed`:
    ```bash
    # cifras = 12345
    echo -n $cifras | sed 's/./&+/g; s/.$//' | bc
    # el primer bloque s/./&+/g añade un "+" despues de cada dígito. 
    # $cifras = "1+2+3+4+5+".
    # el segundo bloque s/.$// elimina el ultimo "+".
    # $cifras = "1+2+3+4+5".
    # bc es una calculadora cientifica, que calculará la suma.
    ```
    Explicación:
    - `s/./&+/g`: Esta expresión regular s en sed busca cualquier carácter (.) en la cadena y lo reemplaza por sí mismo (&) seguido de un signo más (+). El & en la parte de reemplazo representa el dígito encontrado en la cadena original.
    - `s/.$//`: Esta expresión regular s en sed busca el último carácter (.$) en la cadena y lo reemplaza por una cadena vacía, lo que efectivamente lo elimina.
## Práctica 2: Disco duro, particiones
### Creación de disco
Creamos un disco nuevo con la interfaz del vm virtualbox:

Para comprobar que esta bien, lsblk y sale el nuevo disco llamado sdb.
- Abrimos configuración → Almacenamiento → Nuevo disco duro → vdi → 1gb.
### Particiones y Filesystem
Ahora vamos a particionar el disco con gdisk en 5 particiones de 200mb (la ultima tamaño auto). 
```bash
gdisk /dev/sdb
# con p nos enseñan las nuevas particiones
# con n hacemos una nueva particion
# con w guardas las particiones
```

Ahora hay que dar formato a las particiones:
```bash
mkfs -t reiserfs /dev/sdb1
mkfs -t xfs /dev/sdb2
mkfs -t ext2 /dev/sdb3
mkfs -t ext2 /dev/sdb4
mkfs -t ext4 /dev/sdb5
```
### Apt-get install / cache
Para descargar reiserfs hacemos:
```bash
apt-get install reiserfs
# no sale na
```
Para buscarlo en otros repositorios:
```bash
apt-cache search reiserfs
```
Si no funciona, haz `apt-get update` y se vuelve a intentar.
Si sigue fallando, hay que cambiar el servidor dns, haciendo `nano /etc/resolv.conf`, y ponemos `nameserver 8.8.8.8.`

y al ver uno que se parece reiserfsprogs, y se hace despues `apt-get install reiserfsprogs`.

hacemos lo mismo para el xfs con el `apt-cache`, y nos salen muchos,` | more `para ver mas.
es el xfsprogs.

Ahora montamos las particiones, para lo que hacemos 5 carpetas en `/`.

Con el mount montamos los discos:
```bash
mount /dev/sdb1 /disco1
mount /dev/sdb2 /disco2
...
mount /dev/sdb5 /disco5
```

Con el comando `df -h` vemos los discos montados y en que directorio se encuentran. 
Vemos que el reiserfs ocupa ya el 17% de la particion. El xfs ocupa un 6%. 
Por lo general siempre vamos a dar el formato ext4.

Luego miramos el `/etc/fstab` para hacer los cambios permanentes.

metemos:
```bash
/dev/sdb1	/disco1	reiserfs	defaults	0	0
/dev/sdb2	/disco2	xfs	defaults	0	0
/dev/sdb3	/disco3	ext2	defaults	0	0
/dev/sdb4	/disco4	ext2	defaults	0	0
/dev/sdb5	/disco5	ext4	defaults	0	0

```

Al salir, hacemos `mount -a` para saber si hay algun problema (si no sale nada esque va bien).
### Home en disco duro externo
Ahora vamos a meter el home en el disco 5.
```bash
mkdir /temporal # Creamos dir nuevo
umount /dev/sdb5 # Desmontamos el quinto disco
mount /dev/sdb5 /temporal # Lo montamos en el nuevo dir
cp -r /home/* /temporal # Este método no guarda los permisos
```
sale lost+found pero da igual, en temporal y en home hay lo mismo. 
Sin embargo, alumno no tiene permisos en la copia!. 

Para corregirlo, hacemos:
```bash
cp -rpa /home/* /temporal # Ahora conserva permisos
```
Así conserva los permisos el propietario original.
```bash
# Para dejar todo como estaba.
umount /dev/sdb5
rmdir /temporal
```
### Sustituir tres particiones por una que ocupe tamaño total
Tenemos sdb1,sdb2,sdb3, y queremos hacer sdb1 que ocupen las 3.

En `gdisk`:
Borramos la partición 1 2 y 3, y hacemos una nueva partición con el tamaño de las anteriores (tamaño por defecto).

Aún así, el filesystem ocupa 200M, lo que ocupaba la partición 1.

Para ello, se podría dar nuevo tamaño a la nueva particion haciendo resize, y así no perdemos datos originales.

En este caso, no tenemos nada, así que hacemos:
```bash
 mkfs -t ext3 /dev/sdb1
```
Así el filesystem ya ocupa toda la partición y se puede utilizar en su totalidad (comprobar con `df -h` o `lsblk`).
- Si no se amplía el tamaño:
    ```bash
    umount /dev/sdb1
    resize2fs /dev/sdb1 # te pide que hagas el e2fsck -f
    e2fsck -f /dev/sdb1
    resize2fs /dev/sdb1
    mount /dev/sdb1 /ruta/de/montaje
    df -h # comprobar que tiene el tamaño deseado
    lsblk # de las dos formas vale
    ```

## Práctica 3: Usuarios
### Archivos de usuarios, grupos y contraseñas

Los grupos de usuarios se encuentran en `/etc/group`

El formato es:
```bash
nombreGrupo : x : gId (idGrupo) : usuarios que forman parte del grupo :
```

Ahora tenemos que crear un usuario manolito con gid 5000

Los usuarios se almacenan en `/etc/passwd`

El formato es:
```bash
nombreUsuario : x : uId (idUsuario) : gId : GECOS (nombreCompleto,numeroHabitacion,tlfTrabajo,tlfCasa) : directorio home : consola predeterminada
```

Las contraseñas se almacenan en `/etc/shadow`
El formato es:
```bash
nombreUsuario : contraseñaEncriptada : 19381 (dias que han pasado desde 1 de enero de 1970) : 0 (dias hasta que se tenga que cambiar la contraseña, 0 = desactivado) : (maximo de dias para cambiar la contraseña) : (numero de dias para recibir un warning de que cambies la contraseña) : (dias inactivos para que el usuario caduque) : (fecha de expiración)
```
### Creación de usuarios

Para cambiar los GECOS (informacion adicional al usuario) usamos el comando 
```bash
chfn manolito
```

Para crear un usuario los pasos son:
1. Crear su grupo en /etc/group
2. Crear el usuario en etc/passwd
3. Crear su directorio home y darle propiedad a manolito con chown y a su grupo con chgrp
    - Nota: si no tiene .bashrc en el directorio, es bueno copiarlos del /etc/skel con `cp /etc/skel/.* /home/manolito`, que copia lo oculto pero sin el `-r`, que hace recursivo y copia lo que hay en el directorio anterior. Para hacer que esos archivos tambien pertenezcan a manolito hacemos `chown -r` y `chgrp -r` con el directorio /home/manolito para darle permisos a su grupo.
4. Asignarle una contraseña (2 maneras):
    - Con el comando passwd (mala idea para hacer scripts porque te pide la contraseña dos veces).
    - Otro más adaptado para scripts.
### Cambiar a contraseña segura
Para obligar a que se cambie de contraseña a una segura:
```bash
chage -m 90 -d 0 -E 1/1/2025
# Siendo m: en 90 dias se cambia (periódico).
       # d: para que cambie en el dia 0 (al iniciar sesión).
       # E: fecha de expiración.
```
### John the Ripper
Ahora usamos *John the Ripper* para obtener las contraseña:
```bash
# instalacion del john
apt-get install john 
# se crea un archivo que combina los usuarios y las contraseñas
unshadow /etc/passwd /etc/shadow > /contrasenha
# se usa ese archivo con el formato crypt
john --format=crypt /contrasenha
```
Con esto te lo va crackeando poco a poco.

Ahora usamos *libpam* para protegerte del john the ripper, limitando el tamaño de las contraseñas y dandole numero de retrys.
```bash
apt-get install libpam-cracklib
```
En `/etc/pam.d/common-password` tiene la configuración, para las limitaciones.

### Borrado de usuarios
Para borrar un usuario:
```bash
userdel -r manolito # -r para borrar todo lo relacionado
                    # con el usuario en concreto.
```
Para ver si hay algo de ese usuario:
```bash
find / -user manolito
```
Para crear un usuario
```bash
useradd manolito
```
### Creación de scripts con AWK
Ahora creamos un script que genere usuarios y contraseñas basados en la información de los alumnos.

Descargamos una Lista.txt
```bash
wget www.ce.unican.es/SI/LabFiles/Lista.txt --user=alumno --password=alu.SI
```
Este archivo contiene nombres, apellidos y correos de los alumnos:
```bash
G663;G663 - Sistemas InformÃ¡ticos

AHEDO GARCIA, DANIEL;dag649@alumnos.unican.es
ALAMO GARCIA, FRANCISCO;fag954@alumnos.unican.es
ARGUMOSA ARROYO, GUILLERMO;gaa18@alumnos.unican.es
AZCORRETA FERNANDEZ, FERNANDO;faf715@alumnos.unican.es
...
FERNANDEZ DE LA MORA, CARLOS;cfd333@alumnos.unican.es
FERNANDEZ LOPEZ, EMILIO;efl281@alumnos.unican.es
FUENTE ALONSO, PABLO DE LA;pfa858@alumnos.unican.es
...
VICENTE ORTIZ, PABLO;pvo688@alumnos.unican.es
YU, KAIYUE;kyu880@alumnos.unican.es
ZALACAIN GARCIA, JUAN;jzg982@alumnos.unican.es
ZAMORA MARTINEZ, RAUL;rzm74@alumnos.unican.es
```
La idea es formar usuarios tomando el primer apellido y añadiéndole al final la primera letra del nombre, y para la contraseña tomar las tres primeras letras del nombre y concatenarles las tres primeras letras del apellido.

Con el primer usuario quedaría de la siguiente forma:
```bash
------------------
alumno: AHEDO GARCIA, DANIEL
nombre: DANIEL
apell.: AHEDO
------------------
user: AHEDOD
contra: DANAHE
------------------
```
El script es el siguiente:
```bash
#!/bin/bash

read linea < Lista.txt
read linea < Lista.txt
while read linea
do
	TEXTO=$(echo $linea | awk -F';' '{print $1}')
	NOMBRE=$(echo $TEXTO | awk -F', ' '{print $2}')
	APELLIDO1=$(echo $TEXTO | awk '{print $1}')
	APELLIDO1=$(echo $APELLIDO1 | awk -F',' '{print $1}')
	LETRA=$(echo $NOMBRE | awk '{print substr($1,1,1)}')
	USER=$APELLIDO1$LETRA
	NOMBRE3=$(echo $NOMBRE | awk '{print substr($1,1,3)}')
	APELLIDO3=$(echo $APELLIDO1 | awk '{print substr($1,1,3)}')
	CONTRA=$NOMBRE3$APELLIDO3
echo ------------------
echo alumno: $TEXTO
echo nombre: $NOMBRE
echo apell.: $APELLIDO1
echo ------------------
echo user: $USER
echo contra: $CONTRA
done < Lista.txt
```

Script para crear usuarios a partir de un archivo:

```bash
#!/bin/bash
while read linea
do
nombre=$(echo $linea | awk -F ';' '{print $1}')
user=$(echo $linea | awk -F ';' '{print $2}')
uid=$(echo $linea | awk -F ';' '{print $3}')
gid=$(echo $linea | awk -F ';' '{print $4}')
shel1=$(echo $linea | awk -F ';' '{print $5}')
password=$(echo $linea | awk -F ';' '{print $6}')
cat /etc/passwd | grep $user

if [ $? -ne 0 ]
then
    echo el usuario $user no existe
    adduser --gid $gid --uid $uid --shell $shell --gecos "$nombre",,, $user 
    echo $password | chpasswd $user
else
    echo el usuario $user existe
fi

# echo $nombre
# echo $user
# echo $uid
# echo $gid
# echo $shell
# echo $password
done < usuarios.txt
```

Script que busca archivos inmovibles (con una flag i puesta, se ve con lsattr y se cambia con chattr)
```bash
#!/bin/bash
USUARIO=$1
cat /etc/passwd | grep $USUARIO > /dev/null
if [ $? -ne 0 ]
then
    echo el usuario no existe
    exit
fi
echo usuario: $USUARIO

Isattr /home/$USUARIO | grep -e '-i-' | awk '{print $2}' > temporal.txt 
while read archivo
do
    echo $archivo | grep .temp > /dev/null
    if [ $? -eq 0 ] # es .temp
    then
        echo borraria $archivo
        #rm $i
    fi
    echo $archivo | grep .fix > /dev/null 
    if [ $? -ne 0 ] # entonces es .fix
    then
        echo chattr -i $archivo
    else # no es ni .temp ni .fix
        echo dejo $archivo como esta
    fi
done < temporal.txt
rm temporal.txt
```


## Práctica 4 - Software, descarga de archivos

Queremos instalar el paquete openssh en nuestro linux.

3 formas:

1. Codigo fuente y compilado (archivos .tar)
2. Repositorios
3. Paquetes precompilados


### 1ª forma de instalación software: Código fuente y compilado (archivos .tar)

El programa: `openssh`

Donde lo queremos instalar: `/opt/openssh/<version>`

Abrimos esto en el navegador (fuera de linux)
_http://www.openssh.org/portable.html_

Y buscamos una descarga que nos guste (alemania berlin), el cual nos referencia a este link: 	_https://ftp.spline.de/pub/OpenBSD/OpenSSH/portable/_

Aquí buscamos el más actual que sea _tar.gz_ (el resto es información). Click derecho y copiar enlace:
Ahora descargamos el archivo con wget
```bash
wget https://ftp.spline.de/pub/OpenBSD/OpenSSH/portable/openssh-9.6p1.tar.gz
```

Ahora desempaquetamos y descomprimimos:
```bash
tar -xzvf nombre.tar.gz
# x: extraer
# z: descomprimir
# v: verbose
# f: file
```

Se ha creado una carpeta con el nombre openssh-9.6p1 en el directorio en el que estabamos.

Para saber si se puede instalar (comprobar sus dependencias) ejecutamos el archivo configure (o config o parecido): 
```bash
./configure --prefix=/opt/openssh/9.6
```
Dice que le hace falta el `zlib`.
```bash
# probamos a instalarlo
apt-get zlib
# no existe, hay que buscarlo
apt-cache search zlib | more
```
Sale una lista grande y no sabemos cual es el paquete correcto. Probando algunos paquetes, acertamos con el paquete _zlib1g-dev_`_.

Hacemos `apt-get install zlib1g-dev` y tras ello, volvemos a hacer el configure. Esta vez tarda más pero encuentra otra dependencia, `libcrypto`.

Volvemos a instalarlo con el `apt-cache libcrypto`. Volvemos a probar, y con _libssl-dev_ y al acabar volvemos a ejecutar el configure.

Ahora acaba sin fallos, por lo que vamos a ejecutarlo.
```bash
make # esto compila el programa
make install # esto lo instala
```

### 2ª forma: Repositorios y llaves (keys)

En `/etc/apt/sources.list` están los repositorios de donde buscan los apt-get. En el caso de chrome añadimos la línea:
```bash
deb http://dl.google.com/linux/chrome/deb   stable  main
```

Si te piden las llaves o keys → `apt-key list` te muestra todas las keys del llavero. Para añadir llaves `apt-key add`.
En el caso de chrome:
```bash
wget https://dl.google.com/linux/linux_signing_key.pub
apt-key add linux_signing_key.pub
apt-cache search chrome
# no sale
apt-get update
apt-cache search chrome
apt-get install google-chrome-stable
```

Para ejecutar un programa en cualquier directorio (por ejemplo, ls) se debe añadir su path completo (directorio que lo contiene) en la variable de entorno $PATH.

Instalamos nedit
```bash
apt-get install nedit
```

Al hacer nedit, no funciona porque necesita entorno gráfico.
Hay que descargarse las _xwings_.


Descargamos el entorno gráfico:
```bash
apt-get install xorg
```

Para iniciar: `startx`.

Se abre una consola, escribimos nedit y funciona el entorno gráfico.

### 3ª forma: Paquetes precompilados (.deb) 

Descargamos stress.rpm con wget (paquete de fedora) y queremos ejecutarlo en debian. Para ello descargamos `alien` para ejecutarlo en debian:
```bash
apt-get install alien
alien stress.rpm
```
Y se genera stress.deb.

Con dpkg instalamos paquetes precompilados
```bash
dpkg -i stress.deb
```
Si nos equivocamos para borrar y empezar de cero.
```bash
dpkg -purge paquete.deb # borra todo el paquete
```
Para encontrar el archivo `whereis stress.deb` y nos sale en usr/bin/stress. Si le queremos encontrar con el comando find : `find / -name stress*`

Para scripts, si un comando ha tenido exito la variable `$?` es 0, si ha fallado `$?` es 1 o similar.

Para saber si nedit está instalado:
apt list nedit | grep instalado
echo $?


## Práctica 5 - Redes
Clonamos el core_si enlazado y con macs reiniciadas (ejemplos)
Ahora creamos 2 nats:
- nat 1: 10.0.2.0/24
- nat 2: 10.0.3.0/24

Ahora el core si al nat 1 y el server si al nat 1 y 2.

Lo primero hay que cambiar de nombre (hostname) en `/etc/hostname`. 
Luego hay que cambiar las interfaces, en `/etc/network/interfaces`.
El server tiene dos interfaces, pero no están habilitadas. Al hacer `ifconfig -a` nos muestra todas las interfaces. Ahí sale enp0s8 desactivada, la tenemos que activar más tarde. También nos piden apagar el loopback, para eso hacemos `ifconfig lo down`.
Para volverlo a encender `ifconfig lo up`.
Ahora encendemos enp0s8 de manera permanente:
```bash
.
.
.
allow-hotplug enp0s8
iface enp0s8 inet dhcp
```
Si queremos hacerlo estatico:
```bash
.
.
.
allow-hotplug enp0s8
iface enp0s8 inet static
address 10.0.3.4
netmask 255.255.255.0
network 10.0.3.0
broadcast 10.0.3.255
```
para saber el gateway y mas información hacemos `route -n` que nos muestra la tabla de rutas.

Para cambiar el gateway al enp0s8, hay que borrar la linea de 0.0.0.0 a 10.0.2.1 con `route del -net 0.0.0.0`

y para poner el enp0s8
`route add default enp0s8`

Ahora queremos que tambien las 10.0.2.0 vaya por el s8, para eso borramos la ruta en la tabla: `route del -net 10.0.2.0 netmask 255.255.255.0`

Añadimos la ruta 193.144.198.0:
`route add -net 193.144.198.0 netmask 255.255.255.0 gw 10.0.2.1 enp0s3`

Para hacer una conexión a una pagina web, metes la dirección web, que un dns te lo convierte en una ip. En `/etc/hosts` tenemos una lista de "direcciones" con sus ips. Si queremos añadir alguno a la lista, en vez de esperar al dns, lo resuelve directamente (DNS Spoofing).

Ahora miramos el `/etc/resolv.conf`
Tenemos los servidores dns que busca la maquina.

Cuando metes una pagina web la maquina va a hacer:
- Busca si la direccion web está en /etc/hosts
- Llama a un dns de /etc/resolv.conf

## Práctica 6 - Arranque

Fases de arranque:
- Fase 1, Arranque Hardware o arranque EFI: Antes del arranque de SI, se ejecuta el grubx64.efi. Sin él, no arranca el kernel.
Se arranca el EFI:

Para escribir ':' → 'shift' + 'ñ'

Para escribir '/' → '-'


Nos metemos en /EFI/debian y buscamos el grubx64.efi (lo han podido cambiar de nombre o de sitio) si le encontramos, lo ejecutamos poniendo el nombre.

- Fase 2: Cargador del Kernel o GRUB:

En la carpeta /boot/grub/ abrimos el archivo grub.cfg
Bajamos y vemos las configuraciones de arranque de distintos kernels y con el modo discovery en las lineas menuentry.

Si le borramos y luego hacemos update-grub, se genera el grub.cfg. En general, cada vez que hacemos un cambio, se tiene que hacer update-grub.

En la carpeta /etc/grub.d

Instalamos refind (otro cargador del kernel) y ahora queremos abrir con el grub, no con el refind. Para usar el grub antes que el refind tenemos que cambiar el orden del Efi Boot Manager, para verlo `efibootmgr`.    
Para cambiar el orden `efibootmgr -o 4,7,0,1,2`


- Fase 3: Cargar del Kernel:

En /boot encontramos vmlinux.5.10.0-20-amd64 y vmlinux.5.10.0-21-amd64, asociados a initrd.img-5.10.0-20-amd64 y initrd.img-5.10.0-21-amd64 (son la version del kernel actual y el pasado, por si falla el actual.)

Cuando arrancas te sale el menu azul, con tres opciones:
La primera es la de siempre, luego el advanced te deja arrancar eligiendo que kernel quieres, incluyendo el modo recovery (arranque sin servicios). 
Si en vez de entrar con un enter, le das a la tecla 'e', te sale un script en el que editamos el arranque.
Si en vez de entrar con un enter, le das a la tecla 'c', se abre una consola del grub (otra vez en teclas inglesas):
Hacemos un ls y nos salen las unidades (como los discos duros)
nos metemos al (hd0,gpt2) que es donde tenemos el /.

Para ver el interior ls (hd0,gpt2)/boot/ (Así vemos si los archivos han cambiado de nombre o lo que sea)

Para probarlo, cambiamos el vmlinuz-5.10.0-21-amd64 a vmlinux-5.10.0-21-amd64. Ahora casca, tenemos que buscar una forma de arreglarlo. Podríamos usar el kernel antiguo (el -20),pero vamos a arreglar el arranque de este. Nos metemos a advanced options, y donde te deja elegir los kernel, le damos a la tecla 'e'. Ahí buscamos la línea "linux /boot/vmlinuz-5.10..." y lo cambiamos por el nombre que habiamos puesto, "linux /boot/vmlinux-5.10...". Hacemos ctrl + x y ya arranca, pero este cambio es solo de una vez. Una vez iniciamos, cambiamos el nombre al correcto.


En /boot hay varias cosas:
- /efi/EFI/debian: varios archivos, buscamos los .efi:
    - grubx64.efi

## Práctica 6: Servicios


Con systemctl ves todos los servicios.

Con el `systemctl status cron` vemos el estado (active running).
Hacemos `systemctl stop cron` y comprobamos el estado (inactive dead).

Descargamos un servicio con un wget (check-disk-space.sh)

Los servicios se almacenan en `/usr/lib/systemd/system`

Abrimos el cron.service:
Tiene esta estructura
```bash
[Unit] // info sin más
Descripcion
Man
After // dependencias de servicios, significa que este servicio se ejecuta después de los indicados en esta clausula
Before // significa que este servicio se ejecuta antes de los indicados en esta clausula
[Service]
EnvironmentFile
ExecStart // Es el comando que se ejecuta al ejecutar el servicio
[Install]
WantedBy=multi-user.target //El target es como la configuracion de servicios, un target tiene x servicios y otro target tiene otros. por defecto es multi-user.target
```
Creamos una copia del cron llamado miservicio.service en 
`/usr/lib/systemd/system`.

Para que esté en un target, tenemos que hacer un link en `/etc/systemd/system/multi-user.target.wants` a mano, lo cual es un coñazo.

Para hacerlo más rapido, hacemos un `systemctl enable miservicio.service`, que crea el link.

Para arrancarle `systemctl start miservicio.service`

Si en un servicio tiene en el unit wants es que necesita un servicio, pero si se enciende el servicio no enciende el mencionado en el wants.

Por otro lado, si en el unit hay un servicio con el requires, si se apaga el servicio que se menciona,pero si se enciende el servicio enciende el requires porque le hace falta para funcionar.

Mientras que los servicios se ejecutan una vez justo al final del arranque, los timer se ejecutan una vez cada tiempo determinado.

Para hacer un timer en la carpeta `/usr/lib/systemd/system`, siempre es un .timer y un .service asociado.

Queremos hacer un servicio con timer para que el check-disk-space.sh se ejecute cada minuto. Para ello, creamos un miservicio.service y un miservicio.timer.
#### _check-disk-space.sh_
```bash
#!/bin/bash
MAX=95
EMAIL=root@localdomain
PART=$1
USE=`df -h |grep $PART | awk '{ print $5 }' | cut -d'%' -f1`
if [ $USE -gt $MAX ]; then
	echo "Percent used: $USE" | mail -s "Running out of disk space" $EMAIL
else
	echo "------------ `date` ------------" >> /var/log/check-disk.log
	echo " Disk usage: $USE %" >> /var/log/check-disk.log 	
fi

```
#### _miservicio.service_
```bash
[Unit]
Description=Este es mi servicio
Requires=cron.service
[Service]
ExecStart=/usr/bin/sh /root/check-disk-space.sh /dev/sda2 

[Install]
WantedBy=multi-user.target
```
#### _miservicio.timer_
```bash
[Unit]
Description=Timer de mi servicio

[Timer]
OnBootSec=1
OnUnitActiveSec=60

[Install]
WantedBy=timers.target
```

Para activarlo, hacemos un `systemctl enable miservicio.service` y `systemctl enable miservicio.service`, que crea el link.

También podemos cambiar de target: - Con `systemctl set-default rescue.target` entramos en el modo rescate.
- Con `systemctl set-default multi-user.target.wants` entramos en el multi user (por defecto está este).


## Práctica 7: Ficheros Avanzados

Discos físicos separados que hacemos que el sistema operativo los vea juntos.

Descargamos lvm2, que es una librería que permite hacer volumenes lógicos.

Creamos 2 discos de 2gb cada uno y partimos cada uno en dos particiones de 1GB. Ahora buscamos unir las dos particiones del primer disco y una tercera del segundo disco.

Para ello creamos un volumen fisico (PV) con:
```bash
pvcreate /dev/sdb1
pvcreate /dev/sdb2
pvcreate /dev/sdc1
pvdisplay
```
Ahora creamos un volumen de grupo, que agrupará estos tres volúmenes físicos, llamado vg0.

```bash
vgcreate vg0 /dev/sdb1 /dev/sdb2 /dev/sdc1
vgdisplay
```
Después creamos un volumen lógico o disco llamado lv0

```bash
lvcreate -l 100%FREE -n lv0 vg0
# -l que coja 100% del espacio disponible
# -n le da el nombre
lvdisplay
```
Al hacer `lvdisplay` nos sale en LV Path /dev/vg0/lv0, que es el path al volumen que hemos creado. (Se podría particionar con el gdisk haciendo `gdisk /dev/vg0/lv0` y actuaría como un disco normal).

Nos piden meter la carpeta /boot en este disco virtual, pero para ello primero hay que darle formato con `mkfs` y luego lo montamos en un directorio nuevo:
```bash
mkfs -t ext4 /dev/vg0/lv0
mkdir /disco
mount /dev/vg0/lv0 /disco
# el disco ha sido montado
cp -r /boot /disco
ls disco
```

Ahora queremos ampliar el volumen logico para que incluya la cuarta particion:

```bash
pvcreate /dev/sdc2
vgextend vg0 /dev/sdc2
vgdisplay # deberia indicar que hay 2gb
lvextend -l +100%FREE /dev/vg0/lv0
lvdisplay # sale 4GiB.
df -h # sale 2,9GiB
# Hay que extender el filesystem
resize2fs /dev/vg0/lv0
df -h # sale 3,9GiB
```
### Paridad y redundancia
Para hacer paridad o redundancia usaremos RAID, que utilizan distintos metodos de conservacion de información.

El mirroring hace una copia exacta de un disco en otro (pero no es muy efectivo porque pierdes mucho espacio, para escribir 2gb necesitas 4gb).

Otro sistema es el bit de paridad, usando 4 discos, se almacena info en 3 y la paridad en el 4.

Otro sistema es el stripping, que escribe en paralelo en varios discos.

Backups

Backup 0: Hace una copia de todo sin importar si hay otros backups.

Backups incrementales: Copia las modificaciones desde el último backups

Para hacer un disco con stripping usamos mdadm (apt-get install mdadm)
Creamos 4 discos de 512MB y le hacemos una particion a cada uno con gdisk, n (todo por defecto) y luego w.
Luego creamos esa union de esos discos con el mdadm
```bash
mdadm --create /dev/md0 --verbose --level=5 --raid-devices=4 /dev/sda1 /dev/sde1 /dev/sdf1 /dev/sdf1

cat /proc/mdstat # con esto ves el estado del disco creado por varios
mdadm /dev/md0 -f /dev/sde1 # quitamos uno de los discos

cat /proc/mdstat # vemos que falla el sde1
```

Ahora le quitamos fisicamente y creamos otro disco exactamente igual.
Entramos (si el md0 se ha cambiado el nombre a md127 para arreglarlo update-initramfs -u)
Para poner otro disco hacemos: 
`mdadm /dev/md0 -a /dev/sde1` 
Si según lo hacemos, hacemos el `cat /proc/mdstat` nos sale el porcentaje de recovery, pero lo hace bastante rápido.

Ahora hacemos un disco de 2GB, y le hacemos 2 particiones de 1GB.

Entramos en alumno (contraseña temporal) y hacemos dos .txt.
Desde root, creamos el directorio temporal y copiamos /home con cp -rp /home /temporal y borramos /home. Creamos home otra vez y montamos el disco sdh1 en /home, y el sdh2 en un nuevo directorio /backups, todo en el /etc/fstab.

Queremos hacer un backup de /home en el segundo disco:
```bash
dump -0u -f /backups/backuphome /home # 0u porque es de nivel 0, es decir, el que más ocupa.
cat /var/lib/dumpdates # log que muestra todos los backups realizados
```

Ahora desde alumno creamos un archivo nuevo `hola > ficheroNuevo.txt` y desde root hacemos un nuevo backup, pero esta vez de nivel 1 (un backup incremental, dependería del 0):

```bash
dump -1u -f /backups/backuphome1 /home # 0u porque es de nivel 0, es decir, el que más ocupa.
```

Si quieres comprobar lo que hay en el backup y navegar por el `restore -i -f /backups/backuphome`

Para restaurarlo primero vamos a borrar un archivo y luego nos movemos a /home (el / del backup) y hacemos `restore -x -f /backups/backuphome` y salen unas preguntas del volumen (o nivel) en el que está (le damos un 1).

## Práctica 8: Recursos

`ps` te dan los procesos activos
`ps -aux` te da todos los procesos, los secundarios
`ps -aux | wc -l` dice cuantas lineas (cuantos procesos)
`ps -aux | grep root | wc -l` dice cuantos procesos tiene root
para saber cuanto tienes de espacio en cada disco `df -h | awk ...`
para saber cuanto tienes de espacio en general (y en la swap) `free`

Las facilities son grupos de servicios que logean cosas juntas (parecido a los target con el tema de iniciarse, pero en este caso crean y modifican los logs en el mismo fichero, esto se encuentra especificado en el `/etc/rsyslog.conf`)


Para logear en el journal hacemos `systemd-cat -p warning -t local0 'echo Hola'` y para encontrarlo en el journal hacemos `journalctl | grep Hola`


Para ver que se logea en cada fichero log vamos al fichero `/etc/rsyslog.conf`

Para buscar que hay entre dos fechas en el journal usamos `journalctl -S "2024-10-5" -U "2024-10-10"`

Para buscar que hay desde hace una semana en el journal usamos
`journalctl -S "1 week ago"`

Para buscar que hay desde hace una hora en el journal usamos
`journalctl -S "1 hour ago"`


Vamos a usar una terminal multiple con `tmux`:
Dentro, usamos CTRL + B, y después SHIFT + 5 (%) y se nos abren dos terminales horizontales.
y con CTRL + B, y después SHIFT + 2 (") y se nos abren dos terminales verticales.
Y con CTRL + B y las flechas, cambiamos las pantallitas.

Queremos dos pantallitas abajo y una arriba.
En la de arriba, top.
En la de abajo a la izq, su alumno. En la otra root.

En `/sys/devices/system/cpu/cpu1/online` tenemos la opcion de apagar una cpu, si ponemos un 0, en el top pondrá la medicion arriba de absent.

Cambiamos htop por top

Corremos dos stress con `stress -c 1 &` y lo vemos en el htop

Desde el root usamos el comando renice, que te cambia el nice (o lo contrario a la prioridad) su valor maximo es -20 y su menor 19.
En /etc/security/limits.conf metemos alumno hard cpu 1, con eso te deja usar la cpu durante un minuto



## Práctica ?: Kernel
Para saber el kernel que usamos `uname -a`

para descargarlo apt-get install linux-source-5.1

y te lo descarga en `/usr/src`. Para descomprimir 
```bash
tar xfv linux-source-5.1.tar.xz
cd linux-source-5.1
make defconfig # crear fichero de config
make bzImage # compila el vmlinuz
make modules
make modules_install
make install # copia en /boot update-grub
```

descargamos unos modulos del repositorio de unican con el wget.

Para reconfigurar de forma dinamica el kernel se usa el sysctl -w dev.cdrom.autoeject=1 y así cambiamos los valores del kernel, pero no es permanente.
Para verlos todos sysctl -a.

Para hacer que haga reboot en 15 cuando kernel panic hacemos sysctl -w kernel.panic=15

Para que sea permanente, se cambia en /etc/sysctl.conf


## Exámenes de teoría
### Examen parcial 2023

- Cuestión 1 (1p). Describe en qué consisten la redirección y las tuberías (pipes) en la shell, para
qué se usan y sus diferencias.
    - Las pipes _(|)_ se usan para hacer que la salida de un comando sea la entrada del siguiente, pudiendo así encadenar procesos hasta alcanzar el resultado deseado. Sin embargo, las redirecciones redirigen la entrada o salida de un proceso a un archivo concreto, haciendo posible almacenar la salida de un programa _(proceso > archivo)_, los errores de un programa _(proceso 2> archivo)_, acumular varias salidas en el mismo archivo _(proceso >> archivo)_, o darle un archivo de entrada a un proceso _(proceso < archivo)_.

- Cuestión 2 (1p). En un ejercicio de la práctica de sistemas de ficheros trabajábamos con un disco
de 1GB en el que hacíamos 5 particiones de 200MB cada una. Si sabemos que parte del espacio
en disco es ocupado por las estructuras GPT (GPT Header primario y secundario), ¿Es posible
crear esas 5 particiones con el tamaño indicado? ¿Por qué?.
    - Las _estructuras GPT_ son cabeceras que dan información sobre las particiones. El GPT header primario va al principio del disco, y el GPT header secundario es redundancia del primero pero está al final. Probablemente no se pueda llegar a hacer 5 particiones de 200MB, pero si se pueden hacer 4 de 200 y una ultima del espacio restante.

- Cuestión 3 (1p). Describe qué es un extent, en qué tipo de sistema de ficheros se utiliza y cuáles
son las ventajas e inconvenientes de su utilización.
    - Un _extent_ es un bloque de información usado en los sistemas de ficheros ext y optimizan mucho el almacenamiento de datos, guardando unicamente el sector de inicio y de final, haciendo el redireccionamiento mucho más asequible.
    Al usar este mecanismo, se evita la fragmentación, y se optimiza el acceso a datos adyacentes, ya que los datos contiguos están relacionados entre sí.

- Cuestión 4 (1.5p). En el sistema de ficheros ext4 cada directorio tiene asociado un i-nodo en la
tabla de i-nodos, con un puntero a al menos un bloque de datos. Describe el contenido de los
bloques de datos de un directorio.
    - ???

- Cuestión 6 (1.5p). En la figura inferior se muestra el contenido del fichero /etc/passwd de tu
máquina. Indica, como administrador, qué cambios harías en su contenido.
    ```bash
    root::0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:
    daemon:x:2:2:daemon:/sbin:
    adm:x:3:4:adm:/var/adm:
    lp:x:4:7:lp:/var/spool/lpd:
    …
    user1:x:500:500::/home/test:/bin/bash
    user2:x:500:501::/home/user2:/bin/false
    …
    user100:x:601:601::/home/user100:/bin/tcsh
    user101:602:602::/home/user101:/bin/bash
    ```

    ```bash
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/bin/bash
    daemon:x:2:2:daemon:/sbin:/bin/bash
    adm:x:3:4:adm:/var/adm:/bin/bash
    lp:x:4:7:lp:/var/spool/lpd:/bin/bash
    …
    user1:x:500:500:user1:/home/test:/bin/bash
    user2:x:500:501:user2:/home/user2:/bin/bash
    …
    user100:x:601:601:user100:/home/user100:/bin/tcsh
    user101:602:602::user101/home/user101:/bin/bash
    ```
### Examen parcial 2022

- Cuestión 1. Describe qué son las variables de entorno, así como la forma que tiene un usuario
de crear sus propias variables de forma permanente. Una de las variables importantes es $PATH. Describe su función y qué pasa si se borra su contenido original.
    - Las _variables de entorno_ son variables que el sistema operativo crea para guardar información importante, configuraciones o ubicaciones relevantes. En el caso del _$PATH_ se almacenan los directorios donde se encuentran los comandos o procesos utilizados por los usuarios en la terminal. Suelen ser el _/bin, /usr/bin y /sbin_. En caso de que se borrase, no se podrían usar comandos escribiendo en la terminal tal cual, se tendrían que proporcionar los path concretos a cada ejecutable.

- Cuestión 2. En las capturas de pantalla inferiores observamos el análisis temporal de dos
mecanismos alternativos para mover un fichero (de tamaño 512MB). Justifica las diferencias
observadas en el resultado.
    ```bash
    [ (SI) root core ~] time (cp bigfile.txt /home/test/ ; rm bigfile.txt)
    real 0m2,216s
    user 0m0,008s
    sys 0m1,360s
    [ (SI) root core ~] time (mv bigfile.txt /home/test/)
    real 0m0,001s
    user 0m0,000s
    sys 0m0,000s
    ```
    -El _move (mv)_ siempre va a ser más rápido puesto que es solo cambiar el puntero de ubicación a otro directorio (sin necesidad de manipular los datos), mientras que usar el _copy (cp)_ copia de una ubicación de memoria uno o más archivos para después recrearlos en otro directorio.

- Cuestión 3. El apagado descontrolado (“botonazo”) de nuestro equipo puede causar problemas
de consistencia en nuestro sistema de ficheros. Indica por qué se producen dichos problemas, qué
efectos tienen y cuáles son las medidas defensivas del sistema de ficheros ante este problema.
    - Si se apaga el ordenador de la que se está procesando un cambio en la memoria, se podría interrumpir este proceso. Para solucionar esto, muchos sistemas operativos llevan a cabo el _journaling_, una metodología que hace que antes de tomar una acción en memoria antes se apunte en un _journal_ y luego se realice la acción. Si se apaga el ordenador de la que estaba realizando la acción, en el siguiente arranque se revisa el journal y si hay algún cambio sin concluirse, se acaba en el arranque.

- Cuestión 4. En el sistema de ficheros ext4 uno de los cambios principales consiste en sustituir el
sistema previo de punteros (12 punteros directos + 3 indirectos) por “extents”. Indica en qué
consiste dicho cambio, así como su principal ventaja.
    - Los _extents_ son bloques de almacenamiento que se almacenan en memoria indicando su primer y último sector. Esto hace que sea menos propenso a fragmentar la memoria y hace que los datos que se encuentran en el mismo extent esten relacionados, por lo que hace que los accesos adyacentes sean más efectivos.

- Cuestión 5. El fichero /etc/shadow almacena una copia encriptada de las claves de los usuarios.
Si no existe un algoritmo capaz de desencriptar las claves, ¿es un riesgo de seguridad que los
usuarios tengan acceso de solo lectura a dicho fichero? (Justifica tu respuesta)
    - Tener el archivo shadow expuesto para los usuarios es un riesgo de seguridad grave, puesto que no tiene una funcionalidad para los usuarios más allá de ver la contraseña encriptada, ya que existe el archivo /etc/passwd que muestra lo mismo pero sin la contraseña, siendo mucho más seguro para la integridad del sistema.

- Cuestión 6. Describe el problema de persistencia existente con la asignación de nombres para los
dispositivos de almacenamiento (/dev/sdx) y las alternativas existentes para darle solución.
    - El problema que presenta este sistema el nombre asignado a x unidad puede variar entre reinicios, puesto que en cada arranque se asocian los nombres. Para solucionar esto se le pueden dar nombres fijos asociados al UUID (esto se puede introducir en el /etc/fstab).

### Examen Parcial 2021

- Cuestión 1 (2p) Describe el problema de funcionamiento que supone para tu Shell la ejecución
del siguiente comando: (export PATH=/home)
    - Esto cambia el $PATH a home, por lo que ahora todos los comandos que quieras usar en la terminal deben estar almacenados en el directorio /home y, en caso contrario, deberán ser invocados con su path completo, puesto que el resto de directorios contenidos en la variable $PATH han sido sustituidos por /home.

- Cuestión 3 (2p) Describe el sistema de permisos de ficheros/directorios para un FS tipo ext4 (el
que utilizamos en la asignatura)
    - Los permisos que se usan en linux vienen dados por 3 niveles:
        - Usuario: El usuario propietario del archivo.
        - Grupo: El grupo de usuarios al que pertenece el archivo.
        - Resto: El resto de usuarios que no tiene nada que ver con el archivo.
    - Estos 3 niveles pueden tener distintos tipos de permisos:
        - Lectura (Leer el archivo)
        - Escritura (Modificar el archivo)
        - Ejecución ("Usar" el archivo)  

    Estos tipos se almacenan en flags de 9 bits, 3 bits para cada nivel, y dentro del mismo indicando el primer bit el permiso a lectura, el segundo de escritura y el tercero de ejecución (0 = no permiso, 1 = permiso).

- Cuestión 5 (2p) ¿Debe un administrador conocer las contraseñas de todos los usuarios de la
máquina que administra? 
    - No, debe conocer sus contraseñas y proteger las contraseñas del resto de usuarios, siguiendo la ley del mínimo privilegio.

### Examen Final 2023

# Ejercicio 2
Estamos en la consola GRUB, tenemos que buscar el grub.cfg
```bash
ls (hd0,gpt2)/ # particion 2 del disco, correspondiente a /
# lo vemos ahí

root=(hd0,gpt2)
linux /vmlinuz root=/dev/sda2 # update-grub
initrd /initrd
boot
```

Ahora una vez entramos ya colocamos los archivos donde tocan, en /boot

# Ejercicio 4
Si queremos todos los dias excepto los findes de semana a las 12 de la noche

Hacemos un script:
```bash
#!/bin/bash

anho=$(date | awk -F ' ' '{print $4}')
mes=$(date | awk -F ' ' '{print $3}')
dia=$(date | awk -F ' ' '{print $2}')

FILE=$anho-$mes-$dia.dump

dump -iu -f /opt/backup/$FILE /home/user4

```
Para usar el cron:
```bash
#   min     hora        diadelmes      mes   diadelasemana     comando
    0       0           *               *    2-6                 /bin/bash /root/script.sh

```

O por servicios y timer:

miservicio.service
```bash
[Unit]
Description=Regular background program processing daemon

[Service]
EnvironmentFile=-/etc/default/cron
ExecStart=/bin/bash /root/script.sh
[Install]
WantedBy=multi-user.target
```
miservicio.timer
```bash
[Unit]
Description=Timer de mi servicio
[Timer]
OnCalendar=Mon..Fri 00:00:00
[Install]
WantedBy=timers.target
```
# Ejercicio 5

mkdir /temporal
cp -rpa /home/* /temporal
umount /home

gdisk /dev/sdc → hacemos 1 particion
gdisk /dev/sdd → hacemos 1 particion

Hacemos physicalvolume

pvcreate /dev/sdb1
pvcreate /dev/sdc1
pvcreate /dev/sdd1

Hacemos virtualgroup

vgcreate vg1 /dev/sdb1 /dev/sdc1 /dev/sdd1

Hacemos logicalvolume

lvcreate -l 100%FREE vg1 -n lv1

Ahora ya tenemos un disco logico enorme

Lo montamos en el /etc/fstab al home y luego le metemos el contenido de temporal con cp -rpa /temporal/* /home

Luego te piden esto: El reparto de recursos se hace a través de cuotas, en las que el usuario2 dispone del 50% del espacio de
almacenamiento y el resto de usuario se reparten el restante de manera lo más equitativa posible.

Para hcer esto hay que activar cuotas, en /etc/fstab, en vez de defaults, ponemos usrquota (si fuese de grupos grpquota).
