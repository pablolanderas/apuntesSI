# Practica 2

#### Índice

- [Añadir un disco a la maquina virtual](#add_disc_maq)
- [Inodos](#inodos)
- [Crear archivos random](#crear_arch_random)
- [Particiones](#particones)
    - [Hacer una particion del disco](#hacer_part)
    - [Tabla tamaño de particiones](#tabla_tam_part)
    - [Fijar sistema de ficheros](#fijar_sis_ficheros)
    - [Montar la particion](#montar_particion)
    - [Fichero de montado de particiones](#fich_mont_particiones)
    - [Desmontar un disco](#des_disc)
    - [Juntar particiones](#juntar_part)
    - [Redimensionar particiones](#redimenisonar_part)
    - [Comandos de interres](#comd_interes)
- [Encriptacion y desencriptacion](#encriptacion_desencriptacion)
    - [Encriptacion de archivo](#encriptacion_arch)
    - [Desencriptacion de archivo](#desencriptacion_arch)
    - [Encriptacion de particion](#encriptacion_part)

## Añadir un disco a la maquina virtual <a id="add_disc_maq">

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

## Inodos <a id="inodos">

Para comprobar el inodo de un archivo o directorio se puede utilizar el ``ls`` con la opcion ``-i``. Ten encuenta, si quieres ver todos los archivos de un direcctorio puedes usar ``-ia``

Tambien puedes ver informacion ineresante sobre un archivo con el comando ``stat``

## Crear archivos random <a id="crear_arch_random">

De la siguiente manera puedes crear un archivo random de un tamaño

        dd if=/dev/urandom of=<nombre> bs=<medida> count=<tamaño>

Debes de rellenar el nobmre, el tamaño y la medida, la medida recomendaría poner uno de los siguientes valores:

- 1b: bytes
- 1k: kilobytes (1024 bytes)
- 1M: megabytes (1024 kilobytes)
- 1G: gigabytes (1024 megabytes)
- 1T: terabytes (1024 gigabytes)

## Particiones <a id="particones">

 Los discos apareceran como ficheros en ``/dev/``, el disco principal es ``sda``, y seguido iran ``sdb``, ``sdc``, ...

### Hacer una particion del disco <a id="hacer_part">

Para crear la particion hay que entrar al menu con ``gdisk``. Dentro de este puedes buscar ayuda con ``?``

Para crear la nueva particion se utiliza ``n`` seguido se siguen los siguientes pasos (si no escribes ningun valor se escribe el por defecto):

1. El numero de la particion: Por defecto, la siguiente que toque, al empezar la 1
2. El comienzo del sector: Por defecto el comienzo o en caso de haber una particion despues de esta, pero se puede escribir la direccion exacta en la que quieres que empiece, o lo mas util utilizar + y - para seleccionar cuanto quiere que avance o retroceda en tamaño, mirar como escribir el tamaño en la [tabla de tamaños](#tabla_tam_part)
3. El final del sector: Por defecto el final del disco, pero se puede utilizar de igual forma que la manera anterior, si se utiliza el + sirve para seleccionar el tamaño de la particion
4. El tipo de particion: Dejarlo por defecto

Seguido para salir guardando los datos escribimos el comando ``w``

Con esto  has creado la particion, pero faltan varias cosas por hacer ya que solo has reservado el espacio en memoria, falta [fijar un sistema de ficheros a una particion](#fijar_sis_ficheros), [montar la particion](#montar_particion) y establecerlo en el [fichero de montado de particiones](#fich_mont_particiones)

### Tabla tamaño de particiones <a id="tabla_tam_part">

Este es un ejemplo de como hacer las medidas con 100 de cada caso

| Medida    | Forma de escribir |
|-----------|-------------------|
| Kilobytes | +100K             |
| Megabytes | +100M             |
| Gigabytes | +100G             |
| Terabytes | +100T             |
| Petabytes | +100P             |

### Fijar sistema de ficheros <a id="fijar_sis_ficheros">

Para fijar el sistema de ficheros a una particion se utiliza el siguiente comnado:

        /sbin/mkfs -t <fstype> <particion>

- fstype: Es el sistema de ficheros (por defecto usar ``ext4``)
- particion: Es la direccion de la particion (/dev/particion)

Hay algunas particiones que no estan instaladas, en caso de necesitarlas se pueden instalar de la siguiente forma:

| Sistema de ficheros | Descarga                  |
|---------------------|---------------------------|
| reiserfs            | apt install reiserfsprogs |
| xf                  | apt install xfsprogs      |

### Montar la particion <a id="montar_particion">

Motar la particion asigna la particion a un directorio, esto hace que se pueda acceder a los datos. Ten en cuenta que cuando montas una particion en un sistema de ficheros, no se podra acceder a los ficheros que habia antes, ya que funciona como un puntero y cuando accedes a la carpeta te llevara a la particion si quieres volver a acceder a los archivos es tan sencillo como [desmontarlo](#des_disc).

Antes de montar una particion debes de haberle [fijado el sistema de ficheros](#fijar_sis_ficheros)

Para montar una particion se hace con el siguiente comando:

        mount <particion> <dir_montado>

### Fichero de montado de particiones <a id="fich_mont_particiones">

Este es un fichero que monta todas las particiones cuando inicias la maquina. Si quieres que un montado sea "permanente" tienes que añadirlo al fichero. 

El fichero se encuentra en *``/etc/fstab``*

La estructura del archivo es la siguiente:

    <particion> <punto_mont> <sis_fich> <opc_mont> 0 2

- *particion*: La direccion de la particion, pero preferiblemente es mejor poner el identificador (se obtiene de [lsblk](#lsblk)), si se va a hacer de esta forma escibir ``UUID=<identificador>``
- *punto_mont*: El direcctorio donde se va a montar el fichero
- *sis_fich*: El tipo de sistema de ficheros
- *opc_mont*: Las opciones de montado, ponder ``defaults``

### Desmontar un disco <a id="des_disc">

Para desmontar el disco de un directorio simplemente hay que escribir el siguiente comando:

        umount <directorio>

### Juntar particiones <a id="juntar_part">

Para juntar vairas particiones se pueden eliminar con [``gdisk``](#hacer_part) y crear una en el espacio de todas.

Despues crear la nueva particion con el tamaño de todas la que hay que juntar, hay que [asignarle el sistema de ficheros](#fijar_sis_ficheros) y por ultimo [redimensionar la particion](#redimenisonar_part)

### Redimensionar particiones <a id="redimenisonar_part">

Para redimensionar particiones se usa `*``resize2fs``*. Para usarlo, el disco debe de estar desmontado ([desmontar disco](#des_disc)) y se llama a la funcion de la siguiente forma:

        resize2fs <particion>

Se pueden ver las configuranciones que se pueden usar con ``man`` pero alguna util es:

- -M: Redimensiona la particion al tamaño nimino permitido

### Comandos de interres <a id="comd_interes">

- ``lsblk``<a id="lsblk">: Muestra la informacion de las particiones (con ``-f`` ves informacion extra)

## Encriptacion y desencriptacion <a id="encriptacion_desencriptacion">

### Encriptacion de archivo <a id="entriptacion_arch">

Para encriptar un archivo se utiliza el siguiente comando:

        openssl enc -aes-256-cbc -salt -in <archivo> -out <salida>

En el comando te pedira la contraseña con la cual se podra desencriptar el archivo

### Desencriptacion de archivo <a id="desencriptacion_arch">

Para desencriptar un archivo se utiliza el siguiente comando:

        openssl enc -d -aes-256-cbc -in <archivo> -out <salida>

Durante la ejecucion te pedira la contraseña y saldra el archivo descifrado correctamente

### Encriptacion de particion <a id="encriptacion_part">

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