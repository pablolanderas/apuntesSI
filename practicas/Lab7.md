### 1
Crear los disctos

### 2

Crear las particiones

a)

    apt-get install lvm2

    pvcreate /dev/sdb1

    pvcreate /dev/sdb2

    ....

    vgdisplay -v # Para ver info

Crear el volumen de grupo

    vgcreate vg01 /dev/sdb1 /dev/sdb2 /dev/sdc1 # <vg01> es el nombre del grupo

Crear el volumen logico

    lvcreate vg01 -l 100%FREE -n lv0 # <lv0> es el nombre del volumen

    -L: Para meter tamaño
    
    -l: para meter porcentaje

    lvdisplay 

    mkfs -t ext4 /dev/vg01/lv0

    mount /dev/vg01/lv0 /disco

Añadir particion a un volumen lógico

    vgextend vg01 /dev/sdc2

    lvextend -l +100%FREE /dev/vg01/lv0

    # Rehacer el tamaño de la particion, mirar apuntes

    resize2fs /dev/vg01/lv0

Hacer una RAID

    hacer todos los: gdisk /dev/sdd .....

    apt-get install mdadm

    mdadm --create /dev/md0 --verbose --level=5 --raid-devices=3 /dev/sdd1 /dev/sde1 /dev/sdf1

    </dev/md0>: El nombre de la RAID
    <level>: El niver que te pide (RAID 5)
    <raid-devices>: 

    # Le asignamos el sistema de ficheros y lo montamos, y lo hacemos permanente con el UUID ya que cambia el nombre al arrancar

Recuperar un disco

    Cargarte el disco: mdadm /dev/md0 -f /dev/sdd1

    Eliminarlo: mdadm /dev/md0 -r /dev/sdd1

    Recuperarlo con el g: mdadm /dev/md0 -a /dev/sdg1

Backups

    # Creamos la particion y lo hacemos permanente

    dump -0u -f /backups/copia /home 



# APUNTES DE YAGO

Crear disco de 2GB (sdh)  
$gdisk /dev/sdh  
hacer 2 particiones de +1G el 1º y el resto por defecto  
$mkfs -t ext4 /dev/sdh1  
$mkfs -t ext4 /dev/sdh2  
METER /HOME EN OTRO DISCO  
$mkdir /temp  
$mount /dev/sdh1 /temp  
$cp -rp /home/* /temp  
copiamos todo el home en el disco  
$ls -l /home 
$ls -l /temp comprobar que es igual  
$umount /dev/sdh1  
$rm -r /home  
$mkdir /home  
$nano /etc/fstab  
añadir: /dev/sdh1 /home ext4 defaults 0 0  
$mount -a  
$reboot  
$df -h  
CREAR BACKUP  
$mkdir /backups  
$nano /etc/fstab  
añadir: /dev/sdh2 /backups ext4 defaults 0 0  
$mount -a  
$reboot  
$df -h  
➔ meterse como alumno ($su alumno) en carpeta /home/alumno  
$dd if=/dev/random of=aleatorio count=1 bs=1024k  
$dd if=/dev/random of=aleatorio2 count=1 bs=1024k  
➔ Logearse como root ($su root)  
$dump -0u -f /backups/copia /home  
para hacer el backup de un directorio  
$ls -l /backups  
$cat /var/lib/dumpdates  
aparecen las fechas de backup  
RESTAURAR BACKUP  
$restore -i -f /backups/copia  
herramienta para restaurar  
$ls  
$cd alumno  
$q (para salir)  
➔ Logearse como alumno  
$cat hola > hola  
➔ Logearse como root 
$ls -l bakcups  
$dump -1u -f /backups/copia1 /home  
$cat /var/lib/dumpdates  
aparece el nuevo dump  
$ls -l /backups  
$restore -i -f /backups/copia1  
solo aparece el backup de lo nuevo (hola)  
➔ Cambiar a alumno y eliminar TODO  
rm *  
➔ Volver a root y restaurar alumno  
$restore -r -f /backups/copia  
$ls -l /home /alumno  
$cd /alumno  
ahi esta el contenido que hay que mover al home  
$cd /home  
$restore -r -f /backups/copia  
$ls  
sobreescribe  
