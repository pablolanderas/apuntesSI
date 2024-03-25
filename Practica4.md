# Practica 4

### Indice

- [Intalacion desde codigo fuente](#rep_inst)
- [Instalacion desde paquete](#inst_pack)
    - [Instalar un paquete](#apt_install)
    - [Buscar un paquete](#apt_search)
    - [Refrescar lista de paquetes](#apt_update)
    - [Actualizar los paquetes](#apt_upgrade)
    - [Añadir repositorios](#apt_add_repository)

## Intalacion desde codigo fuente <a id="rep_inst">

Para instalar un software desde codigo fuente hay que seguir los siguientes pasos:

1. **Descargar el software**: Para descargarlo utilizar el comando ``wget`` mas la URL
2. **Descomprimir el archivo**: Una vez descargado el archivo hay que descomprimirlo con el siguiente comando

        tar –xzvf <archivo>
3. **Preconfigurar el makefile**: Mirar informacion en el archivo ``README/INSTALL`` pero suele ser un archivo ``configure`` el cual hay que ejecutar. Si al ejecutar el archivo se le añade el argumento ``--prefix=<dir>`` puedes cambiar el sitio en el que lo instala. Es posible que toque resolver algunas dependencias, para esto se pueden [descargar los paquetes](#inst_rep)
4. **Compilar**: Compilar el paquete con el comando ``make install``

## Instalacion desde paquete <a id="inst_pack">

Es el peor ya que hay que gestionar manualmente las dependencias. Apuntes de las diapositivas (pag 11):

    Command dpkg: packet management in Debian: 
        – Format: dpkg --\<options> [packet]
            Option –i (--install): install a downloaded package.
            Option –r (-P purge): Uninstall a package (purge removes also configuration files)
            Option –c: Shows the content of the package.
            Option –b (--build): Compile a package if it’s source code.
            Option –l(--list): List all the packages available. The second character shows the status of the package: [i-installed], [n-not installed], [c-only configuration files]…

## Instalacion desde repositorio <a id="inst_rep">

### Instalar un paquete <a id="apt_install">

Para instalar un paquete utilizar el siguiente comando:

~~~
apt-get install <nombre_paquete>
~~~

Es paquete tiene que ser el nombre exacto, para eso antes deberias de [buscar los paquetes](#apt_search)

### Buscar un paquete <a id="apt_search">

Puedes buscar el nombre de paquetes con el siguiente comando:

~~~
apt-cache search <paquete>
~~~

Si no te sale el paquete puede ser por dos cosas, porque no has [refrescado la lista de paquetes](#apt_update) o que no has [añadido el respositorio](#apt_add_repository) en el que esta el paquete

### Refrescar lista de paquetes <a id="apt_update">

Para refrescar la lista de paquetes por si no te sale alguno es con el siguiente comando:

~~~
apt-get update
~~~

Comprobar que no sale ningun error (Err) en la ejecucion del comando

### Actualizar los paquetes <a id="apt_upgrade">

Para actualizar los paquetes que tienes instalados usar el siguiente comando:

~~~
apt-get upgrade
~~~

### Añadir repositorios <a id="apt_add_repository">

Para añadir un repositorio nuevo hay que añadirlo al archivo *``/etc/apt/sources.lst``*. Este archivo tiene la siguiente estructura:

~~~
<tipo_archivo> <URL> <distribucion> <componentes>
~~~

Esta te la daran. Despues es posible que el repositorio necesite una clave para usarse. La clave es un archivo, este se descarga con el comando ``wget`` y la URL. Una vez descargado este, hay que ejecutar el siguiente comando para añadir la clave al sistema:

~~~
apt-key add <archivo>
~~~

Con esto se añade el respositorio. Solo faltaria [refrescar los paquetes](#apt_update) y comprobar que no den ningun error.