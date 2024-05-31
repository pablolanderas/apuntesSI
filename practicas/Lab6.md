# Parte 1

### 1.

Cambiar el nobre de /boot/efi/EFI/debian/grubx64.efi por nogrubx64.efi

### 2. 

Entrar al FS0(es como el directorio de arranque) con: FS0:

Volvermos a cambiar el nombre para que funciones

Se puede reiniciar, o ejecutar el achivo con grupx64.efi

### 3.

Mover los *.efi a una nueva carpeta /boot/archivos_efi

### 4.

Descargar el disco duro de arranque

Meterlo en la maquina virtual

Arrancar la maquina virtual y en vez de entrar al FS0, entramos al nuevo FS1 que es el disco de arranque y ejecutar el archivo para arrancar (Si has metido bien el disco este paso no hace falta)

Seleccionar uno cualquiera, y al entrar escribir el 13 para acceder al ``13 - es`` para que el teclado este en español

Desde el disco de arranque se puede acceder al disco principal (sda), siendo sda1 el EFI, sda2 los archivos y sda3 el swap

Movemos el kernel al EFI, para eso hay que montar sda1 y sda2, sacar de sda2 el kernel que son los archivos vmlinuz-4.9.0-4-amd64 y initrd.img-4.9-0-4-amd64 y pasarlos al sda1.

Para econtrar los archivos usamos un find -name "\<nombre>"

Los movemos y creamos el archivo startup.nsh, que va a ser un script que se ejecute nada mas iniciar la maquina. En este script añadiremos lo siguiente:

~~~
FS0:     # PARA ACEDER AL EFI
\EFI\debian\vmlinuz-5.10.0-21-amd64 initrd=\EFI\debian\initrd.img-5.10.0-21-amd64 root=/dev/sda2  # PARA ARRANCAR EL KERNEL
~~~

Comprobar que las direcciones estan bien metidas, el script ha de estar en \EFI, no mas adelante

### 5.

Apagar, hacer una snapshot y volver al estado inicial.

### 6.

Hacer los siguientes comandos e instalar refind
~~~
apt-get update
apt-get install refind
~~~

### 7.

Hay que cambiar el sistema de arranque para que arranquee como antes.

Con el comando ``efibootmgr`` puedes ver el orden de las entradas de arranque

Para volver a ponerlo como antes, tenemos que poner la entrada de ``debian`` primero, para esto ejecutamos el siguiente comando

~~~
efibootmgr -o <el orden>
~~~

Para que funcione correctamente hay que cambiar el orden de debian y rEFInd. Pero poner todas las entradas.

### 8.

Cambiar el nombre de ``/boot/vmlinuz-5.10.0-21-amd64``, y hacemos un ``reboot``

Cuando se te reinicia y no te entra, si le das a la ``c``, te abre el terminal del grub.

Hay que decirle al linux donde esta el kernel y el init, para esto usar los siguientes comandos 

~~~
linux <direccion> root=/dev/sda2           #/boot/.....
initrd <direccion>
boot                                       # Para iniciarlo
~~~

Una vez dentro, volvemos a poner el nombre que es

### 9.

Descargo la foto con el siguiente comando:

~~~
wget --user=alumno --password=alu_SI  https://www.ce.unican.es/SI/Grub/grub.png
~~~

Archivo de configuracion = ``/boot/grub/grub.cfg``
Archivo que establece el tema por defecto = ``/etc/grub.d/05_debian_theme``

Ponemos la foto en la carpeta ``/boot/grub``

Y hacemos un ``update-grub`` para actualizar el tema

Para editar el inicio hay que editar el archivo ``/etc/grub.d/40_custom``

Mirar como editarlo con un menuentry bien

~~~
menuentry <nombre> {
    linux ...
    initrd ...
}
~~~

# Parte 2

RECETA EJERCICIOS

Paso 1: Creacion del servicio

Paso 2: Comprobacion
- a ¿Funciona correctamente? (Enable + Start) 
    - Comprobar su estado (Status)
    - Comprobar que hace lo que tiene que hacer
- b ¿Acaba de forma adecuada? (multi-user.target)
- c ¿Se cumplen las estimaciones temporales? (comprobar after/before) (systemd.analyze)
- d ¿Se cumplen las relaciones de DEPENDENCIA? (requires)

### 1.

### 2.

Comprobar los servicios cargados: ``systemctl --type=service``

Comprobar los targets y servicios activos: ``systemctl --type=target --type=service --state=running``

Las dependencias entre targests: ``systemctl list-dependencies``

### 3.

Los servicios se encuentran en: ``/etc/systemd/system/multi-user.target.wants``





