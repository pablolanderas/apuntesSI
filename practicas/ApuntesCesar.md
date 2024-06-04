# APUNTES SI

## Practica 1

## Practica 2

### Añadir un disco a la maquina virtual 

1. Abrir las propiedades de la maquina virtual
2. Ir a almacenamiento
3. Darle al sibomo del cartucho que al poner el raton dice ``Añadir conexion``
4. Seleccionar disdo duro
5. Hacer click en controlador SATA
6. Darle en el menu a crear
7. Seleccionar ``VDI`` y siguiente
8. No marcar reservar completamente ya que te ocupa toda la memoria del archivo y siguiente
9. Seleccionar la ubicacion del archivo con el nombre y el tamaño
10. Terminar

### Inodos

Para comprobar el inodo de un archivo o directorio se puede utilizar el ``ls`` con la opcion ``-i``. Ten encuenta, si quieres ver todos los archivos de un direcctorio puedes usar ``-ia``

Tambien puedes ver informacion ineresante sobre un archivo con el comando ``stat``

### Crear archivos random

De la siguiente manera puedes crear un archivo random de un tamaño

        dd if=/dev/urandom of=<nombre> bs=<medida> count=<tamaño>

Debes de rellenar el nobmre, el tamaño y la medida, la medida recomendaría poner uno de los siguientes valores:

- 1b: bytes
- 1k: kilobytes (1024 bytes)
- 1M: megabytes (1024 kilobytes)
- 1G: gigabytes (1024 megabytes)
- 1T: terabytes (1024 gigabytes)

### Particiones

 Los discos apareceran como ficheros en ``/dev/``, el disco principal es ``sda``, y seguido iran ``sdb``, ``sdc``, ...

#### Hacer una particion del disco

Para crear la particion hay que entrar al menu con ``gdisk``. Dentro de este puedes buscar ayuda con ``?``

Para crear la nueva particion se utiliza ``n`` seguido se siguen los siguientes pasos (si no escribes ningun valor se escribe el por defecto):

1. El numero de la particion: Por defecto, la siguiente que toque, al empezar la 1
2. El comienzo del sector: Por defecto el comienzo o en caso de haber una particion despues de esta, pero se puede escribir la direccion exacta en la que quieres que empiece, o lo mas util utilizar + y - para seleccionar cuanto quiere que avance o retroceda en tamaño, mirar como escribir el tamaño en la [tabla de tamaños](#tabla_tam_part)
3. El final del sector: Por defecto el final del disco, pero se puede utilizar de igual forma que la manera anterior, si se utiliza el + sirve para seleccionar el tamaño de la particion
4. El tipo de particion: Dejarlo por defecto

Seguido para salir guardando los datos escribimos el comando ``w``

Con esto  has creado la particion, pero faltan varias cosas por hacer ya que solo has reservado el espacio en memoria, falta [fijar un sistema de ficheros a una particion](#fijar_sis_ficheros), [montar la particion](#montar_particion) y establecerlo en el [fichero de montado de particiones](#fich_mont_particiones)

#### Tabla tamaño de particiones

Este es un ejemplo de como hacer las medidas con 100 de cada caso

| Medida    | Forma de escribir |
|-----------|-------------------|
| Kilobytes | +100K             |
| Megabytes | +100M             |
| Gigabytes | +100G             |
| Terabytes | +100T             |
| Petabytes | +100P             |

#### Fijar sistema de ficheros

Para fijar el sistema de ficheros a una particion se utiliza el siguiente comnado:

        /sbin/mkfs -t <fstype> <particion>

- fstype: Es el sistema de ficheros (por defecto usar ``ext4``)
- particion: Es la direccion de la particion (/dev/particion)

Hay algunas particiones que no estan instaladas, en caso de necesitarlas se pueden instalar de la siguiente forma:

| Sistema de ficheros | Descarga                  |
|---------------------|---------------------------|
| reiserfs            | apt install reiserfsprogs |
| xf                  | apt install xfsprogs      |

#### Montar la particion

Motar la particion asigna la particion a un directorio, esto hace que se pueda acceder a los datos. Ten en cuenta que cuando montas una particion en un sistema de ficheros, no se podra acceder a los ficheros que habia antes, ya que funciona como un puntero y cuando accedes a la carpeta te llevara a la particion si quieres volver a acceder a los archivos es tan sencillo como [desmontarlo](#des_disc).

Antes de montar una particion debes de haberle [fijado el sistema de ficheros](#fijar_sis_ficheros)

Para montar una particion se hace con el siguiente comando:

        mount <particion> <dir_montado>

#### Fichero de montado de particiones 

Este es un fichero que monta todas las particiones cuando inicias la maquina. Si quieres que un montado sea "permanente" tienes que añadirlo al fichero. 

El fichero se encuentra en *``/etc/fstab``*

La estructura del archivo es la siguiente:

    <particion> <punto_mont> <sis_fich> <opc_mont> 0 2

- *particion*: La direccion de la particion, pero preferiblemente es mejor poner el identificador (se obtiene de [lsblk](#lsblk)), si se va a hacer de esta forma escibir ``UUID=<identificador>``
- *punto_mont*: El direcctorio donde se va a montar el fichero
- *sis_fich*: El tipo de sistema de ficheros
- *opc_mont*: Las opciones de montado, ponder ``defaults``

#### Desmontar un disco

Para desmontar el disco de un directorio simplemente hay que escribir el siguiente comando:

        umount <directorio>

#### Juntar particiones

Para juntar vairas particiones se pueden eliminar con [``gdisk``](#hacer_part) y crear una en el espacio de todas.

Despues crear la nueva particion con el tamaño de todas la que hay que juntar, hay que [asignarle el sistema de ficheros](#fijar_sis_ficheros) y por ultimo [redimensionar la particion](#redimenisonar_part)

#### Redimensionar particiones

Para redimensionar particiones se usa `*``resize2fs``*. Para usarlo, el disco debe de estar desmontado ([desmontar disco](#des_disc)) y se llama a la funcion de la siguiente forma:

        resize2fs <particion>

Se pueden ver las configuranciones que se pueden usar con ``man`` pero alguna util es:

- -M: Redimensiona la particion al tamaño nimino permitido

#### Comandos de interres

- ``lsblk``: Muestra la informacion de las particiones (con ``-f`` ves informacion extra)

### Encriptacion y desencriptacion

#### Encriptacion de archivo

Para encriptar un archivo se utiliza el siguiente comando:

        openssl enc -aes-256-cbc -salt -in <archivo> -out <salida>

En el comando te pedira la contraseña con la cual se podra desencriptar el archivo

#### Desencriptacion de archivo

Para desencriptar un archivo se utiliza el siguiente comando:

        openssl enc -d -aes-256-cbc -in <archivo> -out <salida>

Durante la ejecucion te pedira la contraseña y saldra el archivo descifrado correctamente

#### Encriptacion de particion

1. Descargar la herramienta

                apt-get update
                apt-get install cryptsetup

2. Crear el contenedor privado para la paticion

                cryptsetup luksFormat <particion>

3. Abrir el contenedor

                cryptsetup luksOpen <particion> <nombre_desbloqueo>

4. Crear el sistema de archivos como explicado anteriormente, pero utilizando la nueva particion que se ha creado:

                /sbin/mkfs -t <fstype> /dev/mapper/<nombre_desbloqueo>

5. Montar la particion

                mount /dev/mapper/<nombre_desbloqueo> <dir_montado>
## Practica 3

## Practica 4

## Practica 5
### Creacion de una red y conectarla

1. Creamos las reded en la pestaña Nat con las direcciones  1.0.2.0 y 1.0.3.0 (simplemente crear y dar el nombre nada mas).
2. Añadimos dos adaptadores red de tipo Nat Red cada uno con uno de las red creadas.
3. Observamos que solo sale la red de 1.0.2.0, porque es la unica configurada.

### Modificacion de una red a estatica
#### Ejemplo statico en 10.0.2.0
La configuramos a estatica es:
 - entramos con ``nano /etc/network/interfaces``
 - cambiamos el dhcp por static
 - y añadimos:
```bash
  address 10.0.2.10
  network 10.0.2.0
  broadcast 10.0.2.255
  mask 255.255.255.0
  gateway 10.0.2.1
```

### Modificacion de una red a dinamica
#### Ejemplo statico en 10.0.3.0 desde 0
La configuramos a dinamica es:
 - entramos con ``nano /etc/network/interfaces``
 - y añadimos:
````bash
  allow-hotplug enp0s8
  iface enp0s8 inet dhcp
````

### Modificacion de la tabla de enrutamiento
#### Eliminar entrada, ej default -> 10.0.2.1
Para borrar ejecutamos el comando ``route del -net 0.0.0.0``.

#### Añadir entrada, ej default -> 10.0.3.1 
Utilizamos el comando ``route add default gw 10.0.3.1 enp0s8``.

#### Añadir entrada estandar, ej 10.0.2.0 -> 10.0.3.1
Utilizamos el comando ``route add -net 10.0.2.0 netmask 255.255.255.0 gw 10.0.3.1 enp0s8``.

### Añadir una entrada a la lista de Hosts
Lo utilizamos para evitar estar realizando una traduccion de dns de manera constante, puesto que al acceder a una pagina antes de realizar la traduccion mira dentro de ``/etc/hosts``.
Para modificar entramos con ``nano /etc/hosts`` y añadimos de esta manera:
````bash
(dir ip)  (dir escrita)
127.0.0.1 www.elpais.es 
````

### Servidor dns de la maquina
La maquina virtual revisa en un numero de servidores donde se encuentran las traducciones dns, esta lista se encuentra en ``/etc/resolv.conf``.
Y para poder modificarlo utilizaremos ``nano /etc/resolv.conf``, con el formato:
````bash
(nombre)  (dir)
nameserver  192.168.183.159
````

### Man in the middle
Para realizar un man in the middle lo que necesitamos es crear de manera normal una red, pero a la hora de poner el **gw** colocaremos la direccion del dispositivo que queremos que vea las conexiones, por ej: 10.0.2.15
```bash
  address 10.0.2.20
  network 10.0.2.0
  broadcast 10.0.2.255
  mask 255.255.255.0
  gateway 10.0.2.15
```

## Practica 6
### Fases del arranque
1.  **Arranque hardware**, el cual se encuentra en el EFI y es el archivo grubx64.efi. Dir: ``/boot/efi/EFI/debian/grubx64.efi``.
  - En caso de fallo nos abrira una shell desde donde podremos arrancalo, entrando primero en **fs0:**, el cual es nuestro disco duro y llegaremos al grubx64.efi o su equivalente y lo ejecutaremos.
  - Para crear una copia del arranque copiaremos esto:
  ```bash
  vmlinuz-5.10.0-20-amd64
  initrd = /initrd.img-5.10.0-20-amd64
  root = /dev/sda2
  ```
  en:
  ```terminal
  cd /boot/efi
  ls
  echo "lo de arriba" > startup.nsh
  cat startup.nsh
  ```
  

  
2.  **Cargador kernel**, al empezar en el menu de grub (el azul), la primera opcion es arranca normal, la segunda es te deja arrancar eligiendo el kernel (incluso con recovery) y la tercera son las opciones del kernel.
  - Dentro del sistema existe un archivo que se encarga del funcionamienro del grub al que accedemos con ``/boot/grub nano grub.cfg``
  - cada cambio exige ejecutar un ``update grub``, el cual genera un nuevi grub.cfg
  - En la carpeta ``/etc/grub.s`` tenemos el directorio del grub, desde donde podemos cambiar informacion dobre el grub. Por ej el ``05_debian_theme`` nos da el aspecto del grub.
  - Para **cambiar el aspecto del grub** metiendo una imagen nos lo haremos mediante:
  ```bash
  cd /boot/grub #necesitamos estar dentro de esta carpeta
  wget https://images7.memedroid.com/images/UPLOADED828/5f45bf890bd33.jpeg
  update-grub
  reboot
  ```
3.  **Cargar kernel**
  - si pulsamos a **esc** en le grub nos sale una terminal para interaccionar con el grub.
  - pulsando **e** nos abre el script con el que se ejecuta el arranque.
  - Por alguna razon que los subnormales del montes y jorge no me quieren decir los archivos vmlinux hacen algo. (creo que son los diferentes kernel).
    - Al cambiarlo nos da el error ``Loading /boot/vmlinux-5.10.0-21-amd64 no encontrado``. Para arreglarlo entramos en los comandos de grub **esc**, para ver el posible error. **Si falla un kernel entraremos en la segunda opcion del grub y tomaremos la otra kernel o dentro de la lista de arranque (e) cambiamos al nombre que nos sale.**
4.  **Cargar servicios**
  - Operaciones con **servicio EJ: cron**
    - ``Systemctl`` nos permite ver los servicios del ordenador
    - ``systemctl status cron`` nos muestra el estado de un servicio cron
    - ``systemctl stop cron`` nos detiene el servicio cron
    - ``systemctl start cron`` nos arranca el servicio cron
    - Para **encontrar** un servicio usaremos: ``find / -name cron.service``, suele estar en ``/usr/lib/systemd/system``
    - Dentro del archivo del servicio con nano vemos:
      - Unit/, que incluye:
        - Descripcion del servicio
        - Documentacion del servicio
        - After: Los que se ejecutan antes q este servicio (dependencias de orden)
        - before: los que se ejecutan despues de este servicio (dependientes en orden)
        - requires: dependencias obligatorias, al arrancar el servicio se arrancan las dependencias
        - wants: como el requiers pero menos restructivo
      - Service/
        - ExecStart: El ejecutable en el q se basa el servicio
      - Install/
  - Vamos a **crear un servicio**:
    - Descargamos el servicio de la direccion con ``wget ...``
    - Lo creamos con nano dentro de la carpeta ``/usr/lib/systemd/system`` con el nombre que creamos.
    - Le añadimos lo que nos interese en las diferentes partes.
    - El ejecutable q nos interese va en la linea ``ExecStart``, con su direccion.
    - Para crear el link vamos a la carpeta ``/etc/systemd/system/multi-user.target.wants/`` y ejecutamos ``systemctl enable miservicio``, al servicio le habiamos llamado miservicio.service.
    - Activamos el servicio con ``systemctl start miservicio.service``
  - **Operaciones con timer**
    - Dentro de un timer encontramos:
      - Unit/:
        - Descripcion
        - Documentacion
      - Timer/:
        - OnCalendar:
        - AccuracySec:
        - Persistent:
        - OnBootSec: que inicie en el sec que le decimos
        - OnUnitActiveSec: el periodo de activacion en sec.
      - Install/:
        - WantedBy: ponermos ``timers.target`` para que forme parte de los timers.
  -  Vamos a **crear un timer**:
    - Descargamos el servicio de la direccion con ``wget ...`` si lo piden
    - Lo creamos con nano dentro de la carpeta ``/usr/lib/systemd/system`` con el nombre del servicio que timea.
    - Le añadimos lo que nos interese en las diferentes partes.
    - El ejecutable q nos interese va en la linea ``ExecStart``, con su direccion.
    - Para crear el link vamos a la carpeta ``/etc/systemd/system/multi-user.target.wants/`` y ejecutamos ``systemctl enable miservicio``, al servicio le habiamos llamado miservicio.service.
    - Activamos el servicio con ``systemctl start miservicio.service``
### **Entrar en modo de rescate**
  - Para entrar usaremos la linea: ``systemctl set-default rescue.target``.
  - Para salir usaremos la linea: ``systemctl set-default multi-user.target``.

## Practica 7

### Crear volumenes virtuales
  - Creamos los discos mediante el menu grafico del VirtualBox
  - Creamos las particiones con ``gdisk``
  - Con el comando ``pvcreate /dev/sdb1`` creamos un volumen en sdb, podriamos hacer lo mismo con otros volumenes cambiando el dev
  - Creamos un grupo virtual con ``vgcreate bg0 /dev/sdb1 /dev/sdb2 /dev/sdc1``.
  - **Si esta bien formado lo veremos en** ``vgdisplay``
  - Despues creamos el Lv con ``lvcreate -l 100%FREE -n lv0 gv0``
  - creamos el archivo de fichreros con ``mkfs -t ext4 /dev/vg0/lv0``
  - Montamos en la carpeta q queramos y ya lo podemos usar
### Aumentar un volumen virtual.
**Ej sdc2**
  - Seguimos los pasos de arriba hasta el ``pvcrate``
  - ampliamos con ``pvextend vg0 /dev/sdc2``
  - ampliamos el lv ``lvextend -l +100%FREE /dev/vg0/lv0``

### Escrituras en paralelo
- Para haver escritura en paralelo usamos el raid 5, utilizmos mirroring, si escribes en un disco escribes en otro,lo malo es que si tienes un disco de 4gb, se te queda en la mitad porque la otra mitad se usa para copiar lo que escribes en la primera 

- otro sistema es bit de paridad, usando 4 discos, se almacena info en 3 y en el 4 el de paridad.

- otro sistema es el stripping, que escribe en paralelo en varios discos.

- otro es backup 0 hace una copia de todo sin importar que haya habido otros, el backup 1 incremental: copia desde le ultimo backup

### Creamos unos discos raideados
1. creamos 4 discos nuevos de 512 MB
2. lo particionamos en una particion que ocupa todo
3. hacemos apt-get install mdamd
4. ejecutamos con ``mdadm --create /dev/md0 --verbose --level=5 --raid-devices=4 /dev/sdX`` siendo X la letra y numero, y hay que hacerlo con todos los discos, en este caso 4 veces.
5. luego formatemos con ``mkfs -t ext4 /dev/md0``
6. creamos un directorio con ``mkdir /discoraid``
7. montamos en **/discoraid** con ``mount /dev/md0 /discoraid``

### Fallos en discos
- Para ver los fallos utilizaremos ``cat /proc/mdstat``
- simulamos un fallo con mdad /dev/md0 -f /dev/sde1
- al volver a chequear veremos como sde1[1]**(F)**
- eliminamos con ``mdadm /dev/md0 -r /dev/sde1``
- apagamos mediante señal de apagado
- eliminamos el disco roto y añadimos uno nuevo del mismo tamaño
- si nos llaman el mdo a md127 realizamos:
  - ``update-initramfs -u``
  - apagamos
- creamos una nueva particion (**Gdisk**)
- comprobamos mdstat
- añadimos nuevo con ``mdadm /dev/md0 -a /dev/sde1``
- comprobamos otra vez.
### Creacion de un backup
1. Montamos home:
   1. ``mkdir /temporal``
   2. ``mount /dev/sdh1 /temporal``
   3. ``cp -rp /home/* /temporal``
   4. ``umount /dev/sdh1``
   5. ``rm -r /home/``
   6. ``mkdir /home``
   7. ``mkdir /backup``
2. Cambiamos el contenido de fstab
   1. ``nano /etc/fstab``
   2. ``/dev/sdh1 /home ext4 defaults 0 0``
   3. ``/dev/sdh2 /backup ext4 defaults 0 0``
3. montamos todo ``mount -a``
4. comprobamos con ``df -h``
5. reboot 
6. comprobamos otra vez
7. realizamos un dump com ``dump -ou -f /backup/backuphome /home``
8. Si creamos un backup incremental con la condicion ``dump -1u ...``solo ponemos lo nuevo.
### Recuperar un backup
- ``restore -i -f /backup/backuphome`` nos mete en una terminal para recuperar las cosas con lo q tengamos en el backup.
- ``restore -x -f /backup/backuphome`` nos extrae el archivo.

### Procesos
- para ver los procesos usaremos el comando ps
- para ver todos usamos ps aux
- Y ya usando tuberias podemos hacer muchas cosas.
- ps aux | grep root | wc -l cuenta los numero de procesos 
- del root. **Si no ponemos grep root tenemos que restar 1 porq cuenta los titulos**
-  el comando ``free -h`` te da la info de la memoria y la swap.
-  El comando ``top`` te muestra una lista de los procesos q se estan ejecutandose
-  el comando ``tmux`` nos permite abrir multiples terminales.
-  ``su alumno`` para iniciar el usuario alumno
-  ``kill -s STOP "PID"`` duerme el proceso
-  ``kill -s CONT "PID"`` duerme el proceso
-  Para cambiar la prioridad utilizamos renice "-10" -p "PID", haciendo que tenga mas prioridad **PR mas bajo**, los "" son para el ejemplo no hay q ponerles.
-  con ulimit -a podemos ver las cosas limitables.
-  PAra limitar a alumno utilizamos el archivo ``/etc/security/limits.conf`` y ponemos: ``alumno hard cpu 1``

### Cron
el comando es crontab -e
El cron es para ejecutar de manera peridoca una tarea, el formato para utilizar el crontab es:
  - min hora dia del mes mes dia del semana comando
el comando crontab -l nos lo lista

## Practica 8

### kernel
- Podemos saber la version de linux que tenemos si usamos ``uname -a``
- Vamos a descargar el kernel de nuevo:
  1. pat-get install linux-source-5.1
  2. cd /usr/src
  3. observamos si se encuentra con ls
  4. descomprimimos con tar xvf linux-source-5.1.tar.xz
  5. entramos en la carpeta con cd linux-source-5.1
  6. make defconfig -> crea el fichero config
  7. make bzImage -> lo compila
  8. make modules
  9. make modules_install
  10. make install


## Comandos interesantes
- **>** redirecciona salidas.
- **touch** crea ficheros.
- **route -n** nos da la tabla de rutas de la maquina.
- **ls -l --sort=time** ordena por tiempo.
- **date | cut -f1,3,2,6 -d** nos muestra los campos 1,3,6,2 y el -d te detecta los campos separados por espacios. 
- Para moverme entre terminales **alt + flecha**
- en un fallo de EFI si usamos **shift + ñ** nos salen **:**
- **shift + 9** es **(**
- **shift + 0** es **)**
