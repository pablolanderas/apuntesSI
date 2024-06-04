# Practica 6

#### Indice



## Fases del arranque <a id="fases_arranque">

El arranque del sistema operativo tiene las siguientes fases:

1. **Hardware + UEFI (Boot Manager)** ([ver](#UEFI))

    Es el primer paso de arranque, carga todo el hardware para  poder empezar a lanzar el resto de procesos    

2. **Bootloader (GRUB)** ([ver](#grub))

    Esta etapa intermedia esta para poder proporcionar un sitema multiarranque y poder configurar el arranque del kernel. La tarea de este proceso es cargar el kernel en memoria principal y que empiece a funcionar

3. **Kernel** ([ver](#kernel))

    Esto sucece despues de que el bootloader ha cargado el kernel y los ficheros ramdisk en memoria. El kernel se descomprime a si mismo, detecta el mapa de memoria, la CPU y las características que admite, arranca la consola para mostrar por pantalla información, checkea el bus PCI, creando una tabla con los periféricos detectados, inicializa el sistema encargado de la gestión de la memoria virtual, incluido el swap, inicializa los drivers de los periféricos detectados (monolíticos o modulares), Monta el sistema de ficheros del directorio root “/” y llama a los procesos de systemd (siguiente paso), PID 1 como padre del resto de procesos.

## UEFI <a id="UEFI">

Este hace 2 cosas:

1. **UEFI Image load:** carga imagenes desde el firmware de la memoria del sistema, aqui se realizan los siguientes pasos:

    1. Inicializa la memoria principal, así como los componentes de hardware requeridos para las siguientes fases.

    2. Carga los controladores de dispositivo desde la memoria flash a la memoria principal (ahora disponible), inicializa todo el hardware requerido (disco (sistema de archivos FAT), monitor, teclado, ...) y registra/utiliza protocolos.

2. **UEFI Boot Manager:** implica la selección y carga del sistema operativo. El UEFI Boot Manager es responsable de gestionar el proceso de arranque y decidir qué sistema operativo o aplicación se debe cargar. Los pasos 

    Aqui se intenta cargar aplicaciones UEFI (.efi files) en un orden predefinido (se puede ver con el comando  ``efibootmgr -v``). Estas alicaciones pueden ser cargadores de sistema operativo, núcleos EFI, controladores adicionales (ext4), shell, GUI, etc.

    Las aplicaciones deben residir en un sistema de archivos definido por UEFI: Sistema de archivos FAT (formato de la Partición del Sistema EFI o ESP).

    Para buscar las aplicaciones y cargadores UEFI consulta la tabla de particiones GPT para identificar la ESP(particion VFAT), lee el archivo objetivo (el .efi) y lo ejecuta. Por defecto en Debian la ruta del archivo es ``/efi/boot/bootx64.ef``, esto desde la shell de UEFI, pero desde el sistema encendido es ``/boot/efi/EFI/debian/grubx64.efi``

    **Si no se utiliza un cargador de arranque** (soporte EFI stub habilitado en el kernel), todos los archivos necesarios para cargar el sistema operativo (kernel, ramdisk, etc.) deben estar disponibles en esta partición

### UEFI Shell <a id="UEFI_shell">

EFI es un pequeño sistema operativo en la placa base. 

Se puede acceder a ella pulsado *ESC* despues de encender.

Muchos comandos son como linux: ``cp``, ``ls``, ``rm``, ``mv``, ``touch``, ...  
Se pueden ver todos [aqui en la pagina 84](https://uefi.org/sites/default/files/resources/UEFI_Shell_Spec_2_0.pdf)

Como en este sistema se utiliza el teclado ingles, aqui puedes comprobar que es cada tecla:

<img src="https://media.geeksforgeeks.org/wp-content/cdn-uploads/20201123134530/qwety-1024x630.jpg">

#### **Error en UEFI Shell** <a id="UEFI_shell_error">

Si entras aqui por un error en el arranque, obtendras una lista las particones GPT. La particion de **arranque es FS0**. Para entrar a una particion por ejemplo a FSO se escribe en la termnial ``FSO:``

Para iniciar el grub hay que encontrarlo y ejecutarlo, normalmente tiene el nombre de ``grubx64.efi``. Los otros archivos que puedes encontrar son: BOOTX64.CSV, fbx64.efi, grub.cfg, mmx64.efi, shimx64.efi

Si no encuentras estos archivos o existe algun problema con el arranque, hay que utilizar un [disco de arranque](#disco_arranque) para volver a guardar los .efi y que se pueda volver a ejecutar.

#### **Disco de arranque** <a id="disco_arranque">

Un disco de arranque es un medio de almacenamiento (como un CD, DVD, USB, o disquete) que contiene todos los archivos necesarios para iniciar un sistema operativo o para realizar tareas de recuperación y mantenimiento del sistema

Lo puedes descargar [aqui](https://www.ce.unican.es/SI/SystemRescueCD/systemrescuecd-x86-5.2.1.iso) con el usuario ``alumno`` y la contraseña ``alu_SI``

Una vez descargado, se añade a la maquina virtual. A la hora de iniciar, habra dos FS, el 0 y el 1. Uno de estos es el disco de arranque, si te equivocas puedes usar ``exit``. Dentro de esto ejecuta el archivo que esta al final de la carpeta para iniciar la reparacion

A la que inicia, hay un momento en el que dice que teclado quieres, si estas atento, puedes escribir *13* y asi podras tener el teclado en español, si no seguiras con el en ingles.

Una vez dentro, se puede acceder al disco principal, que es sda. En sda1 esta el EFI, en sda2 los archivos y en sda3 el swap

Para volver a añadir los .efi, montamos los discos 1 y 2 en dos directorios y en este bucamos los datos para pasarlos y que pueda volver a arrancar (si no encuentras los .efi mirar despues):

~~~bash
mkdir /sda1
mkdir /sda2
mount /dev/sda1 /sda1
mount /dev/sda2 /sda2
cd /sda2
find -name "*.efi"
// Los seleccionamos y los copiamos
cp <archivos.efi> /sda1/EFI/debian
~~~

En caso de no haber encontrado los .efi, se puede arrancar directamente con el kernel. Son 2 archivos y se puede buscar igual, normalemnte son ``/boot/vmlinuz-5.10.0-21-amd64`` y ``/boot/initrd.img-4.9-0-4-amd64``.   
Despues lo pasamos tambien al disco de arranque ``/sda1/EFI/debian``
Una vez hecho esto, reiniciamos y entramos al arranque como en [el primer caso](#UEFI_shell_error). Aqui, llegamos hasta el archivo del kernel y ejecutamos el siguiente comando:

~~~bash
\EFI\debian\vmlinuz-5.10.0-21-amd64 initrd=\EFI\debian\initrd.img-5.10.0-21-amd64 root=/dev/sda2
~~~

Una vez hecho esto, si quieremos que esto sea permanente, se puede crear un script que lo ejecute cuando empiece el arranque de EFI. Para esto hay que crear un archivo una vez arancado en la raiz del EFi para que ejecute ese comando. Hay que crear el archivo ``/boot/efi/startup.nsh`` y colocarle el codigo anterior

### Orden de arranque <a id="ord_arranque">

Para ver el orden de arranque de los efi se puede usar el comando ``efibootmgr`` al que se le puede añadir ``-v`` par a ver mas informacion

Para modificar este orden de arranque se utiliza el comando ``efibootmgr -o <identificadores>``, separando los identificadores por comas

### Gestor de arranque <a id="gest_arranque">

Puedes descargar el gestor de arranque rEFInd con los siguientes comandos.

~~~bash
apt-get update
apt-get install refind
~~~

Una vez acabe de instalar, puedes aceptar que se use por defecto o no

## GRUB <a id="grub">

GRUB (Grand Unified Bootloader) es un gestor de arranque utilizado en muchos sistemas operativos, especialmente en sistemas basados en Linux. Su función principal es cargar el kernel del sistema operativo en la memoria y transferirle el control para que el sistema operativo pueda comenzar a ejecutarse.

### Configuracion <a id="grub_conf">

GRUB lee la configuracion en ``/boot/grub/grub.cfg``. Este archivo no se debería de modificar manualmente, se debería de crear a traves del comando **``update-grub``**. Este comando genera el archivo apartir del archivo [``/etc/default/grub``](#arch_grub) y el directorio [``/etc/grub.d/``](#dir_grub.d).

#### Archivo grub <a id="arch_grub">

Se escriben configuraciones con la siguiente estuctura:

~~~
<Etiqueta>=<Valor>
~~~   

Estos son algunos valores posibles:

- *GRUB_DEFAULT*: entrada del menu predeterminada (comienza por 0). Si quieres utilizar la ultima opcion arracanda puedes usar ``saved``

- *GRUB_CMDLINE_LINUX*: añade opciones a la linea de comandos del kernel para todas las entradas de Linux (se escribe entre "")

- *GRUB_TIMEOUT*: Establece el tiempo (en segundos) que GRUB espera antes de arrancar la opción predeterminada

- *GRUB_DISABLE_RECOVERY*: Si se establece en ``true``, oculta las entradas de recuperación en el menú de arranque de GRUB.

- *GRUB_THEME*: Permite especificar una ruta al archivo de tema de GRUB que se utilizará para personalizar la apariencia del menú de arranque

- *GRUB_BACKGROUND*: Permite poner la ruta de una foto de fondo de pantalla (png, jpg, tga)

#### Directorio grub.d <a id="dir_grub.d">

Aqui encontrarás varios scripts numerados. Cada script tiene un propósito específico y se ejecuta en orden numérico cuando ``update-grub`` es llamado. Estos son algunos de los scripts más comunes y su función:

- *00_header*: Este script configura las variables iniciales y los encabezados del archivo grub.cfg. Incluye configuraciones globales como las opciones de línea de comandos del kernel y la configuración gráfica

- *05_debian_theme*: Este script se encarga de configurar el tema visual de GRUB en sistemas basados en Debian. Define el fondo de pantalla, los colores del texto, y otros aspectos visuales

- *10_linux*: Genera entradas de menu para los kernels detectados en el dispositivo raiz del sistema. Escanea los kernels disponibles en el sistema y crea entradas para cada uno

- *30_os-prober*: ejecuta la herramienta os-prober, que detecta otros sistemas operativos instalados en el sistema (como Windows) y genera entradas de menú para ellos

- *40_custom*: un script reservado para que los usuarios agreguen sus propias **entradas de menú** personalizadas. Es editable y permite la adición de configuraciones específicas del usuario que no son manejadas por los otros scripts. Las entradas tienen el siguiente formato:

    ~~~bash
    menuentry '<nombre>' {                                  # El nombre de la entrada de menu
        set root=(hd0,gpt2)
        linux /boot/vmlinuz-5.10.0-20-amd64 root=/dev/sda2  # El kernel y la direccion del root
        initrd /boot/initrd.img-5.10.0-20-amd64             # El init
    }
    ~~~

### Linea de comandos de GRUB <a id="grub_cmd">

Desde el menu de GRUB puedes pulsar ``c`` para entrar en la linea de comandos de GRUB.

En esta consola si haces un ``ls`` te salda una lista de las particiones disponibles. Si haces el ``ls`` unicamente de la particion solo te va a dar informacion de esta, si quieres ver lo que contiene debes de escribir una ``/`` al final. En ``(hd0,gpt2)/`` se encuentra el directorio raiz y en ``(hd0,gpt1)/`` el efi

Durante el arranque desde esta consola puedes editar las opciones de arranque del kernel ([ver el ejemplo](#ejemplo_grub_cmd)):

- *set root=\<path>*: configura la particion raiz donde se encuentran los archivos kernel y initrd

- *linux \<path-to-kernel> root=\<path>*: carga el kernel de Linux desde la partición configurada

- *initrd \<command>*: Ejecuta un binario especifico en vez de ``/sbin/init`` como proceso de inicio

- *boot*: arranca el sistema

Esto es un ejemplo: <a id="ejemplo_grub_cmd">

~~~bash
set root=(hd0,gpt2)
linux \boot\vmlinuz-5.10.0-21-amd64 root=\dev\sda2
initrd \boot\initrd.img-4.9-0-4-amd64
boot
~~~

## Kernel <a id="kernel">

El archivo del kernel es ``/boot/vmlinuz-5.10.0-21-amd64`` y este tiene 2 fases, la primera en la que carga el [Init Ramdisk](#ramdisk)(``/boot/initrd.img-4.9-0-4-amd64``) y la segunda en la que entra a funcionar el [Systemd](#systemd)

Una vez ya se ha cargado el kernel, se ejecuta el daemon de administracion del sistema ``/sbin/init``. Proporciona un administrador de sistema y servicios que se ejecuta como PID 1. 

Este *init* se encarga de tareas de inicio como: configurar el nombre de la computadora y la zona horaria, verificar el estado del disco, montar sistemas de archivos, configurar interfaces de red, etc

### Init Ramdisk <a id="ramdisk">

El kernel utiliza un ramdisk al arrancar. Un ramdisk funciona como un sistema de ficheros (tmpfs, ramfs) voratil.

El **init ram disk** es una franccion de memoria principal (RAM) usada para agilizar el arranque del sistema con el fin de obtener los elementos necesarios para cargar el kernel en el directorio root. En este ramdisk se realiza un montaje previo del ``initrd``, es un sistema de ficheros temporal para que cuando se ejecute el kernel luego pueda tener acceso al directorio root ``/``. 

El bootloader se encarga de cargar el ``initrd`` en RAM y pasarselo al kernel durante el arranque, despues el kernel lo descomprime y lo monta como un sitema de ficheros temporal. Apartir de ahi, este se utiliza para cargar los controladores de los dispositivos necesarios para reconocer y montar el sitema de archivos raiz real. 

Una vez todo este proceso ha acabado, el ``initrd`` se desmonta y se libera de la memora RAM

## Servicios <a id="servicios">

Los servicios son cargados por el *init*. Estos se pueden encontrar en distintas ubicaciones:

- *Servicios del sistema*: ``/usr/lib/systemd/system/``
- *Servicios personalizados*: ``/etc/systemd/system/``
- *Servicios de paquetes del sistema*: ``/lib/systemd/system/`` 
- *Servicios temporales*: ``/run/systemd/system/``

### Comandos de interes <a id="cmd_servicios">

Con el comando ``systemctl`` puedes hacer muchas cosas con los servicios:

#### Ver los servicios: <a id="cmd_servicios_ver_servicios">

Si no pones nada en el comando este muestra una lista con todos los servicios. Este puede recibir algunos argumentos:

- *--type=<tipo>*: con este argumento puedes especificar que tipo quieres. Puedes ver todos los tipos en la siguiente tabla:

    | Tipo        | Descripción                                            |
    |-------------|--------------------------------------------------------|
    | `service`   | Unidades de servicio (por ejemplo, `apache2.service`). |
    | `socket`    | Unidades de socket (por ejemplo, `cups.socket`).       |
    | `target`    | Unidades de objetivo (grupos de unidades, por ejemplo, `multi-user.target`). |
    | `device`    | Unidades de dispositivo (representa dispositivos de hardware). |
    | `mount`     | Unidades de montaje de sistema de archivos (por ejemplo, `home.mount`). |
    | `automount` | Unidades de montaje automático.                        |
    | `swap`      | Unidades de espacio de intercambio.                    |
    | `timer`     | Unidades de temporizador (utilizadas para activar unidades en un horario específico). |
    | `path`      | Unidades de ruta (vigilancia de archivos o directorios). |
    | `slice`     | Unidades de slice (agrupaciones de recursos de cgroups). |
    | `scope`     | Unidades de alcance (usadas para unidades externas al inicio del sistema). |

- *--state=<estado>*: con este argumento puedes especificar que estado quieres. Puedes ver todos los estados en la siguiente tabla:

    | Estado        | Descripción                                                |
    |---------------|------------------------------------------------------------|
    | `active`      | Unidades activas (actualmente en funcionamiento o cargadas). |
    | `inactive`    | Unidades inactivas (no en funcionamiento).                 |
    | `failed`      | Unidades que han fallado.                                  |
    | `activating`  | Unidades en proceso de activación.                         |
    | `deactivating`| Unidades en proceso de desactivación.                      |
    | `reloading`   | Unidades en proceso de recargar su configuración.          |
    | `plugged`     | Unidades de dispositivo que están enchufadas (solo aplica a unidades de tipo `device`). |
    | `mounted`     | Unidades de montaje que están montadas (solo aplica a unidades de tipo `mount`). |
    | `waiting`     | Unidades en estado de espera (aplica a unidades de temporizador y de ruta). |


Tambien puedes ver el setado de un unico servicio si escribes el comando mas ``status`` y el nombre del servicio que quieres comprobar

#### Iniciar/Detener un servicio: <a id="cmd_servicios_iniciar_detener_servicio">

Para iniciar un servicio se utiliza el comando mas ``start`` y el nombre del servicio que quieres iniciar 
Para detener un servicio se utiliza el comando mas ``stop`` y el nombre del servicio que quieres detener

#### Reiniciar un servicio <a id="cmd_servicios_reiniciar_servicio">

Para reiniciar un servicio se utiliza el comando mas ``restart`` y el nombre del servicio que quieres reiniciar

#### Recargar la informacion de un servicio <a id="cmd_servicios_recargar_servicio">

Recargar la configuración de un servicio sin interrumpirlo sirve para aplicar cambios en la configuración de un servicio en ejecución sin tener que detenerlo y volverlo a iniciar. Esto es útil en situaciones donde la disponibilidad continua del servicio es crítica, ya que minimiza el tiempo de inactividad. Para hacer esto se utiliza el comando mas ``reload`` y el nombre del servicio

#### Habilitar/Deshabilitar un servicio para que arranque automaticamente <a id="cmd_servicios_habilitar_deshabilitar_servicio">

Para que un servicio arranque automaticamente cuando se inicia el sistema, se puede utilizar el comando mas ``enable`` y el nombre del servicio  
Para que un servicio no arranque automaticamente cuando se inicia el sistema, se puede utilizar el comando mas ``disable`` y el nombre del servicio

#### Mostrar las dependencias de un servicio <a id="cmd_servicios_dependencias">

Para mostrar todas las dependencias se puede usar el comando mas ``list-dependencies``. Ademas, tambien puedes poner el nombre de un servicio despues para que te muestre unicamente las dependencias de ese servicio

### Implementacion de servicios <a id="impl_servicios">

Para implementar un servicio hay que generar un archivo de unidad con extension ``.service`` y guardarlo en uno de los directorios que se indican [al principio](#servicios). Este archivo tiene el siguiente formato:

~~~unit
[Unit]
Description=Mi Servicio Personalizado
Wants=network-online.target websockets.target
After=network.target

[Service]
ExecStart=/usr/local/bin/mi_script.sh
Restart=always
User=usuario
Group=grupo

[Install]
WantedBy=multi-user.target
~~~

Se puede usar el archivo ``/usr/lib/systemd/system/cron.service`` como plantilla y ir editnadole, haces una copia y lo modificas.  
Este tipo de archivos tiene 3 partes con diferentes directivas:
- [\[Unit\]](#impl_servicios_unit)
- [\[Service\]](#impl_servicios_service)
- [\[Install\]](#impl_servicios_install)

#### \[Unit] <a id="impl_servicios_unit">

- *Description/Documentation*: Una descripcion breve de lo que hacer el servicio (Description) y su documentacion (Documentation)
- *Requires/Wants*: Lista de unidades que deben estar activas para que esta unidad se inicie (Requires) y lista de unidades que se desea que estén activas, pero no es obligatorio (Wants). Todas estas se separan con espacios entre una y otra
- *Before/After*: Especifica que esta unidad debe iniciarse antes (Before) o despues (After) de las unidades listadas. . Todas estas se separan con espacios entre una y otra

#### \[Service] <a id="impl_servicios_service">

- *Type*: categoriza el servicio según su proceso y comportamiento de demonización. Puede ser:
    - *simple*: tipo por defecto.
    - *forking*: el servicio crea un proceso hijo.
    - *oneshot*: systemd espera a que el proceso finalice antes de continuar con otras unidades.
    - *dbus*: la unidad tomará un nombre en el bus de D-Bus.
    - *notify*: el servicio emitirá una notificación cuando haya terminado de iniciar.
    - *idle*: el servicio no se ejecutará hasta que se despachen todos los trabajos
- *ExecStart*: especifica la ruta y los argumentos del comando que se ejecutará para iniciar el servicio
- *ExecStop*: el comando que se ejecutara cuando se detenga el servicio
- *Restart*: especifica cuando se debe de reiniciar el sevicio, tiene estas opciones:
    - *no* (predeterminado): No se reinicia.
    - *on-success*: Se reinicia solo si el servicio termina con éxito.
    - *on-failure*: Se reinicia si el servicio termina con un error.
    - *always*: Siempre se reinicia, sin importar cómo termine.
- *TimeoutSec*:  especifica el tiempo de espera antes de marcar el servicio como fallido (o forzar su finalización) al detenerlo

#### \[Install] <a id="impl_servicios_install">

- *WantedBy*:  se utiliza para especificar una dependencia similar a la directiva Wants en la sección [Unit]. Cuando una unidad con la directiva WantedBy se habilita, se crea un directorio en la ubicación /etc/systemd/system/[unit].wants. Dentro de este directorio se crea un enlace simbólico que establece la dependencia entre las unidades
- *RequiredBy*: se utiliza para especificar las unidades que requieren la unidad actual. Si la unidad actual se habilita, las unidades enumeradas en la directiva RequiredBy también se habilitarán automáticamente. Establece una dependencia fuerte en la que una unidad es necesaria para el funcionamiento de otra unidad