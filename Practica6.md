# Practica 6

#### Indice



## Fases del arranque <a id="fases_arranque">

El arranque del sistema operativo tiene las siguientes fases:

1. **UEFI Image load:** carga imagenes desde el firmware de la memoria del sistema, aqui se realizan los siguientes pasos:

    1. Se inicializa el firmware, realiza una serie de comprobaciones de hardware y configuraciones. Entre ellas el Power-On Self Test (**POST**)

    2. Se lee la tabla de particiones GPT para identificar la partición EFI System Partition (ESP)

    3. Se cargan la Imágenes UEFI, el firmware UEFI busca en la ESP las aplicaciones y controladores UEFI que necesitan ser cargados. Estos pueden incluir gestores de arranque, controladores adicionales y aplicaciones de firmware específicas

    4. Se ejecutan los controladores UEFI necesarios para inicializar dispositivos adicionales

    5. Se prepara para el gestor de arranque una vez que los controladores y las aplicaciones UEFI necesarias están cargados

2. **UEFI Boot Manager:** implica la selección y carga del sistema operativo. El UEFI Boot Manager es responsable de gestionar el proceso de arranque y decidir qué sistema operativo o aplicación se debe cargar. Los pasos clave son:

    1. Verificación de Opciones de Arranque: El UEFI Boot Manager verifica las opciones de arranque configuradas en el firmware. Estas opciones están almacenadas en variables NVRAM (memoria no volátil de acceso aleatorio) y pueden incluir rutas a gestores de arranque, sistemas operativos y otras aplicaciones UEFI.

    2. Selección de la Opción de Arranque: Basado en la configuración de arranque, el UEFI Boot Manager selecciona una opción de arranque. Esto puede ser configurado manualmente por el usuario en la configuración de firmware o automáticamente según la prioridad de arranque.

    3. Carga del Gestor de Arranque: El UEFI Boot Manager carga el gestor de arranque especificado en la opción de arranque seleccionada. En sistemas Linux, el gestor de arranque común es [GRUB](#grub) (GRand Unified Bootloader).

    4. Transferencia del Control al Gestor de Arranque: Una vez que el gestor de arranque está cargado en la memoria, el control se transfiere desde el firmware UEFI al gestor de arranque. Este paso es crítico porque el gestor de arranque es responsable de cargar el kernel del sistema operativo.

    5. Ejecución del Gestor de Arranque: El gestor de arranque toma el control, presenta las opciones de arranque al usuario (si hay múltiples sistemas operativos o configuraciones de arranque) y finalmente carga y ejecuta el kernel del sistema operativo seleccionado.

## UEFI Shell <a id="UEFI_shell">

EFI es un pequeño sistema operativo en la placa base. 

Se puede acceder a ella pulsado *ESC* despues de encender.

Muchos comandos son como linux: ``cp``, ``ls``, ``rm``, ``mv``, ``touch``, ...  
Se pueden ver todos [aqui en la pagina 84](https://uefi.org/sites/default/files/resources/UEFI_Shell_Spec_2_0.pdf)

## GRUB <a id="grub">

GRUB (GRand Unified Bootloader) es un gestor de arranque utilizado en muchos sistemas operativos, especialmente en sistemas basados en Linux. Su función principal es cargar el kernel del sistema operativo en la memoria y transferirle el control para que el sistema operativo pueda comenzar a ejecutarse.

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

#### Directorio grub.d <a id="dir_grub.d">

Aqui encontrarás varios scripts numerados. Cada script tiene un propósito específico y se ejecuta en orden numérico cuando ``update-grub`` es llamado. Estos son algunos de los scripts más comunes y su función:

- *00_header*: Este script configura las variables iniciales y los encabezados del archivo grub.cfg. Incluye configuraciones globales como las opciones de línea de comandos del kernel y la configuración gráfica

- *05_debian_theme*: Este script se encarga de configurar el tema visual de GRUB en sistemas basados en Debian. Define el fondo de pantalla, los colores del texto, y otros aspectos visuales

- *10_linux*: Genera entradas de menu para los kernels detectados en el dispositivo raiz del sistema. Escanea los kernels disponibles en el sistema y crea entradas para cada uno

- *30_os-prober*: ejecuta la herramienta os-prober, que detecta otros sistemas operativos instalados en el sistema (como Windows) y genera entradas de menú para ellos

- *40_custom*: un script reservado para que los usuarios agreguen sus propias entradas de menú personalizadas. Es editable y permite la adición de configuraciones específicas del usuario que no son manejadas por los otros scripts

### Linea de comandos de GRUB <a id="grub_cmd">

Desde el menu de GRUB puedes pulsar ``c`` para entrar en la linea de comandos de GRUB.

Durante el arranque desde esta consola puedes editar las opciones de arranque del kernel:

- *root=\<path>*: configura la particion raiz donde se encuentran los archivos kernel y initrd

- *linux \<path-to-kernel> root=\<path> [options]*: carga el kernel de Linux desde la partición configurada

- *init=\<command>*: Ejecuta un binario especifico en vez de ``/sbin/init`` como proceso de inicio

- *boot*: arranca el sistema