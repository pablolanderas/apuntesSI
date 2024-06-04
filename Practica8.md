# Practica 8

#### Indice

## Monitorizacion de logs <a id="logs_monitor">

Para monitorizar los logs de los procesos se utiliza el comando ``journalctl``. Ademas, si utilizas algunos argumentos puedes ver mas infomracion:

- *-u <nombre>*: te premite ver los logs de un servicio

- *-t <nombre>*: te premite ver los logs de una **etiqueta**

- *-f*: te permite quedarte a la escucha y mostrar los logs en tiempo real

- *-r*: te permite ver los mas recientes primero, ya que te los muestra en reversa

- *--since "<año>-<mes>-<dia> <hora>:<min>:<sec>"*: te permite ver los logs desde un momento dado

- *_PID=<PID>*: te permite ver los logs de un proceso indicandole el PID

Se pueden realizar cambios en la configuracion del log en el archivo `/etc/systemd/journald.conf`. Una ver modificados **debes de reiniciar** el servicio para que los cambios surjan efecto. Esto se hace con ``systemctl restart systemd-journald``.  
En el archivo puedes modificar valores como:

- *Nivel de prioridad de mensajes que salen por pantalla*: para modificar esto se cambia el valor de ``MaxLevelConsole`` y puede tener los siguientes valores:
    - *emerg (0)*: Emergencia
    - *alert (1)*: Alerta
    - *crit (2)*: Crítico
    - *err (3)*: Error
    - *warning (4)*: Advertencia
    - *notice (5)*: Aviso
    - *info (6)*: Informativo
    - *debug (7)*: Depuración

- *Donde se guarda el jouranl*: para modivicar esto se cambia el valor de ``Storage`` y puede tener los siguientes valores:
    - *volatile*: Guarda los logs en /run/log/journal (se pierde al reiniciar).
    - *persistent*: Guarda los logs en /var/log/journal (se conserva al reiniciar).
    - *auto*: Usa almacenamiento persistente si está disponible, de lo contrario usa almacenamiento volátil.
    - *none*: No guarda ningún log.

- *Límites del tamaño de los ficheros journal*: estas son las opciones mas comunes:
    - ``SystemMaxUse``: El tamaño máximo que puede usar el journal en almacenamiento persistente.
    - ``SystemKeepFree``: El espacio libre que el journal debe dejar libre.
    - ``SystemMaxFileSize``: El tamaño máximo de un archivo individual del journal.
    - ``SystemMaxFiles``: El número máximo de archivos de journal.

## Gestion de procesos <a id="gest_proc">

#### Monitorizar procesos <a id="mon_proc">

Para **monitorizar** los procesos se pueden usar los comandos [`ps`](#utl_cmd) y [`htop`](#utl_cmd). 

#### Logs de procesos <a id="logs_proc">

Ademas se pueden **ver los logs** de los procesos con su PID con ``_PID=`` explicado en [monitorizacion de logs](#logs_monitor)

#### Detener procesos <a id="kil_proc">

Para **matar un proceso** se utiliza el comando ``kill``. Este comando recibe el numero de PID del proceso, este lo que hace es mandar una señal al proceso para terminarlo, por defecto envia la señal `-SIGTERM`, pero la puedes modificar. Si quieres matar el proceso de inmediato sin limpieza puedes pasarle antes del PID `-SIGKILL` o su equivalente mas corto que es `-9`

#### Prioridad de los procesos <a id="proc_prio">

Se puede jugar con la **prioridad de los procesos**, para eso tenemos 2 comandos, `nice` cuando el proceso aun no ha comenzado y `rinice` para procesos que ya se estan ejecutando. Las prioridades van de -20 (mayor prioridad) a 19 (menor prioridad), por defecto estos comienzan a 0.  

Para utilizar `nice` se hace de la siguiente manera: ``nice -n <prioridad> <comando>``, un ejemplo prodia ser ``nice -n 10 tar -czf backup.tar.gz /home/user``

El otro comando, `renice`, puede cambiarle la prioridad a 3 tipos distitos, a un proceso especifico pasandole su PID (``renice -n <prioridad> -p <PID>``), a un grupo de procesos con el ID del grupo (``renice -n <prioridad> -g <grupo>``) o a los procesos de un usuario en concreto (``renice -n <prioridad> -u <usuario>``)

#### Limitacion de recursos <a id="limt_rec">

Para limitar los recursos se utiliza el comando `ulimit`. Se puede usar para establecer o mostrar varios tipos de límites de recursos. Los límites pueden ser "soft" (blandos) o "hard" (duros). Los límites "soft" pueden ser incrementados por el usuario, pero no pueden exceder los límites "hard". Los límites "hard" solo pueden ser incrementados por el superusuario (root). 

Con el comando `ulimit -a` puedes ver los limites actuales, en este te sale los comandos para modificar los valores **(ten cuidado porque los cambios no son permanentes, son loso para la sesion, como se hace eso esta explicado despues)**, pero igualmente aqui dejo algunos:

- *Número máximo de archivos abiertos (file descriptors)*: -n 
- *Tamaño máximo de los archivos que se pueden crear*: -f 
- *Número máximo de procesos por usuario*: -u 
- *Tamaño máximo de la pila*: -s 
- *Memoria máxima residente (en KB)*: -m 
- *Tamaño máximo de core dump (en bloques)*: -c 
- *Tiempo máximo de CPU (en segundos)*: -t 

Esto realiza los limites blandos, para modificar los limietes duros antes hay que ponerle `H`, como en este ejemplo: ``ulimit -Hn 4096``

Para realizar los cambios de forma pemanente te utiliza el archivo ``/etc/security/limits.conf``. Este archivo tiene el siguiente formato
~~~bash
<dominio>    <tipo>    <elemento>    <valor>
~~~
- **dominio:** el dominio indica a quien le afecta, puede ser un usuario (se escribe el usuario directamente), un grupo (@grupo) o  `*` para que sea para todos

- **tipo:** indicar si es ``soft`` o ``hard``

- **elemento:**: los elementos pueden ser los siguientes
    - *core*: Tamaño máximo de archivo core generado en caso de un fallo del programa.
    - *data*: Tamaño máximo de segmento de datos.
    - *fsize*: Tamaño máximo de archivo que el usuario puede crear.
    - *memlock*: Tamaño máximo de bloqueo de memoria en bytes.
    - *nofile*: Número máximo de archivos que el usuario puede abrir.
    - *nproc*: Número máximo de procesos que el usuario puede tener en ejecución.
    - *rss*: Tamaño máximo de conjunto de memoria residente.
    - *stack*: Tamaño máximo de pila.
    - *cpu*: Tiempo máximo de CPU en minutos.
    - *as*: Tamaño máximo de espacio de dirección de proceso (sólo para Linux 2.4 y posterior).
    - *maxlogins*: Número máximo de sesiones de inicio de sesión.
    - *maxsyslogins*: Número máximo de sesiones de inicio de sesión del sistema.
    - *priority*: Prioridad máxima de la tarea (sólo para Linux 2.6 y posterior).
    - *locks*: Número máximo de bloqueos de archivo.
    - *sigpending*: Número máximo de señales pendientes.
    - *msgqueue*: Tamaño máximo de cola de mensajes.
    - *nice*: Valor máximo de prioridad nice permitido al usuario.
    - *rtprio*: Valor máximo de prioridad de tiempo real permitido al usuario.

- **valor:** el valor que se le quiere asignar

## Comandos utiles <a id="utl_cmd">

- **ps**: Este comando muestra información sobre los procesos en ejecución en el sistema. Puedes ver los procesos activos, sus identificadores de proceso (PID), uso de recursos y más. Si le añades `aux` te muestra todos los procesos del sistema

- **htop**: Similar a `ps`, pero con una interfaz de usuario más interactiva y amigable. Proporciona una vista más detallada y en tiempo real de los procesos en ejecución, junto con gráficos de uso de CPU y memoria.

- **vmstat**: Muestra información sobre la actividad del sistema, incluyendo estadísticas de memoria, CPU, E/S (entrada/salida) y paginación. Es útil para monitorear el rendimiento del sistema en general.

- **uptime**: Muestra cuánto tiempo ha estado funcionando el sistema desde su última puesta en marcha, así como la carga promedio del sistema durante los últimos 1, 5 y 15 minutos.

- **strace**: Se utiliza para rastrear las llamadas al sistema y las señales que realiza un proceso. Es útil para depurar problemas de ejecución y comprender cómo interactúa un programa con el sistema operativo.

- **free**: Muestra la cantidad de memoria física y de intercambio (swap) disponible y utilizada en el sistema. Es útil para monitorear el uso de la memoria y diagnosticar problemas de agotamiento de memoria.

- **df**: Muestra el espacio en disco utilizado y disponible en todos los sistemas de archivos montados en el sistema. Es útil para verificar el uso de almacenamiento en disco y para planificar la administración de almacenamiento.

- **du**: Muestra el uso del espacio en disco de archivos y directorios específicos. Puedes usarlo para identificar qué archivos o directorios están consumiendo más espacio en disco.
