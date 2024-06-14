# Practica 7

#### Indice

- [LVM (Logical Volume Manager)](#lvm)
    - [Administador de LVM (lvm2)](#administrador_lvm)
- [RAID](#raid)
    - [Niveles](#raid_lev)
    - [Acciones](#raid_acc)
    - [Crear una raid](#raid_create)
    - [Recuperar un disco](#raid_recuperate)
- [Backups](#backups)
    - [Realizar backup](#blackup_make)
    - [Restaurar backup](#blackup_restore)

## LVM (Logical Volume Manager) <a id="lvm">

Un LVM es una tecnología de gestión de almacenamiento en sistemas operativos Unix y Linux. LVM permite una administración más flexible y avanzada de los discos duros y las particiones en un sistema

Existen varios volumenes de datos:

- **Volumen fisico (PV):** estos son los volumenes formados por los discos o las particiones

- **Volumen de grupo (GV)** estos son un *super disco*, se forman al juntar uno o varios volumenes fisicos

- **Volumen logico (LV):** equivalen a *super particiones*, el sistema de ficheros se crea en un volumen de grupo

### Administador de LVM (lvm2) <a id="administrador_lvm">

lvm2 es una libreria para la administracion de los LVM, para eso hay que instalarla con un [instalador de paquetes](Practica4.md#apt_install). Se puede hacer con los siguientes comandos. **Todos los voluemenes una vez creados se gestionan como una particion y puedes realizar las acciones de la [Practica 2](Practica2.md)**

~~~bash
apt-get update
apt-get install lvm2
~~~

Estos son algunos de los comandos que se pueden utilizar:

- ``vgdisplay -v``: para ver informacion <a id="cmd_ver_info_grupos">

- ``pvcreate <particion/disco>``: con este comando puede crear un volumen fisico de una partición o de un disco que pases como parametro (preferible crear una particion aunque sea del disco entero)

- ``vgcreate <nombreGrupo> <volumenes>``: con este comando puedes crear un volumen de grupo con el nombre que le asignes. El comando recibe todos los volumenes fisicos (que debes de haber creado antes) que quieras añadir al grupo. Estos se escriben separados por un espacio entre volumen y volumen

- ``lvcreate <grupo> -n <nombreLV> -L <tamaño>``: con este comando puedes crear un volumen logico. Recibe el nombre del grupo, el nombre que le quieres asignar al volumen y el tamaño (sin + y -) que quieres asignarle al grupo ([ver como se escriben los tamaños](Practica2.md#tabla_tam_part)). Si en vez de un tamaño, quieres asignarle un porcentaje del disco disponible, hay que sustituir el `-L <tamaño>` por `-l <porcentaje>%FREE`, como por ejemplo `-l 100%FREE`

- ``vgextend <nonmbreGrupo> <particion/disco>``: con este comando se puede añadir un volumen fisico a un grupo ya existene.

- ``lvextend -L <tamaño> <ruta_del_LV>``: este comando se utiliza para *unicamente extender* el tamaño de un volumen logico. Este recibe un [tamaño](Practica2.md#tabla_tam_part), que se puede poner con ``+`` para indicar cuanto aumenta su tamaño o sin el para indicar el tamaño final y la ruta del volumen (*[ver](cmd_ver_info_grupos)*)

- ``lvreduce -L <tamaño> <ruta_del_LV>``: este comando se utiliza para *unicamente reducir* el tamaño de un volumen logico. Este recibe un [tamaño](Practica2.md#tabla_tam_part), que se puede poner con ``-`` para indicar cuanto reduce su tamaño o sin el para indicar el tamaño final y la ruta del volumen (*[ver](cmd_ver_info_grupos)*)

- ``vgreduce <nombreGrupo> <particion/disco>``: este comando elimina un volumen fisico de un volumen de grupo

## RAID <a id="raid">

RAID consigue mejorar las prestaciones y fiabilidad de los discos haciendolos tolerantes a fallos.

### Niveles <a id="raid_lev">

RAID tiene distintos niveles:

- **RAID 0**: el objetivo es mejorar lecturas y escrituras, utiliza el 100% del alamacenamiento disponible. No tolera fallos

- **RAID 1**: el objetivo es mejorar las lecturas, tiene 2 copias completas de la informacion. Como se aplica gran redundancia solo tiene una capacidad del 50%. Puede fallar un disco

- **RAID 4**: un disco almacena informacion de paridad del resto de discos. Mejora las lecturas. Las escrituras tienen cuello de botella por el disco de paridad. Si un disco falla, mediante al disco de paridad se podria recuperar aunque el coste de recuperacion es alto

- **RAID 5**: la infomracion de paridad esta repartida en todos los discos, por lo que ya no existe el cuello de botella.

### Acciones <a id="raid_acc">

Para instalar la libreria para usar RAID debes de hacer lo siguiente:

~~~bash
apt-get update
apt-get install mdadm
~~~

Despues se pueden hacer las siguientes acciones:

#### Crear una raid <a id="raid_create">

Se para crear una raid se utiliza el siguiente comando:

~~~bash
mdadm --create <direccion> --level=<nivel_raid> --raid-devices=<numero_de_discos> <lista_de_discos>
~~~

- *direccion*: la direccion de la raid, se escibe ``/dev/md0`` y se avanza el número en caso de haber mas

- *nivel_raid*: el [nivel de RAID](#raid_lev) que se quiere utilizar, si no especifican utilizar el 5

- *numero_de_discos*: el numero de discos activos que se usara, el resto seran de reserva

- *lista_de_discos*: se escribe la direccion de todos los discos que se usaran separados por espacios, todos deben de ser particiones con [``gdisck``](Practica2.md#hacer_part)

#### Recuperar un disco <a id="raid_recuperate">

Para recuperar un disco hay que eliminar<sub>1</sub> de la RAID el disco que este dañado, despues crear una nueva particion identica con [``gdisck``](Practica2.md#hacer_part)<sub>2</sub>, y añadirlo<sub>3</sub> a la RAID. Para hacer esto hay que usar los siguientes comandos:

~~~bash
mdadm /dev/md0 -r /dev/sdc1
gdisk /dev/sdc
/dev/md0 -a /dev/sdc1
~~~

## Backups <a id="backups">

RAID + journaling no es suficiente para proveer con una disponibilidad del 100%, por lo que se realizan backups. 
- El backup no incremental guarda info ya asegurada, pero aumenta la eficiencia 
de recuperacion de datos de backups lejanos a la fecha actual.  
- El proceso de recuperacion en el backup incremental, restaurar un error que 
tienes el lunes por ejemplo, siendo viernes, es muy costoso al tener que "iterar 
sobre los anteriors backups", pero el proceso de guardado es menos costoso 
cuando se realiza al no tener que guardar tanta cantidad de datos.

### Realizar backup <a id="blackup_make">

Para realizar un backup de un direcotrio se utiliza el siguiente comando:

~~~bash
dump -<nivel> <opciones> <directorio_guardado>
~~~

- *nivel*: indica el nivel de copia de seguridad que se va a hacer. Empieza en 0 y es incremental. Si por ejemplo tienes una copia de nivel 0, puedes sobre escribir esa copia o realizar otra sobre esa copia de nivel 1. Hay hasta un maximo de opciones

- *opciones*: son opciones que puede recibir el comando, estas son algunas:

    - *-u*: actualiza el archivo ``/etc/dumpdates`` con la fecha de la copia de seguridad (solo se puede usar si estas haciendo una copia del sistema de archivos completo no de un subdirectorio)

    - *-f \<directorio>*: indica el directorio donde se guardara el backup

- *directorio_guardado*: el directorio que se va a guardar con la copia de seguridad

Comanod de ejemplo:

~~~bash
dump -0 -uf /ruta/al/archivo/backup.dump /home
~~~

### Restaurar backup <a id="blackup_restore">

Para restaurar un backup ya hecho se utiliza el siguiente comando:

~~~bash
restore <opciones>
~~~

- *opciones*: estas son algunas opcones que puedes utilizar:

    - *-r*: restaura todo el sistema de archivos

    - *-i*: es un modo interactivo con el que te permite seleccionar archivos y directorios especificos para restaurar

    - *-t*: muestra el contenido de la copia de seguridad sin restaurar nada

    - *-f \<archivo>*: especifica el archivo de la copia de seguridad

**Si te da errores** es posible que necesites colocarte con cd en el directorio en el que se encuentra el que se encuentra la copia hecha   