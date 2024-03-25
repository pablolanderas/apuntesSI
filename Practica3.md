# Practica 3

#### Índice

- [Grupos](#grupos)
- [Usuarios](#usuarios)
- [Seguridad del usuario](#user_security)
- [Añadir usuario](#add_user)
- [Eliminar usuario](#del_user)
- [Cambiar contraseña a usuario](#change_password)
- [Estructura de las contraseñas](#struct_password)
- [Permisos de ficheros y archivos](#perm_fich_arch)
    - [Ver permisos](#see_perm)
    - [Cambiar permisos](#change_perm)
- [SUDO](#sudo)
- [Comandos de interes](#comd_interes)

## Grupos <a id="grupos">
...

Los grupos sirven para compartir archivos y recursos, dando una serie de permisos a cada grupo.

Los grupos se encuentran en el archivo *``/etc/group``* y tienen la siguiente estructura

~~~
<grupo>:<contraseña>:<GID>:<usuarios>
~~~

- grupo: Es el nombre del grupo
- contraseña: normalmente esta se encuenta en otro archivo
- GID: el GID del grupo
- usuarios: lista de usuarios del grupo separados por una coma

## Usuarios <a id="usuarios">

Los usuarios se encuentran en el archivo *``/etc/passwd``* con la siguiente estructura:

~~~
<nombre>:<contraseña>:<UID>:<GID>:<descripcion>:<dir_ini>:<shell>
~~~

- nombre: El nombre del usuario
- contraseña: La contraseña del usuario, no se suele configurar aqui si no que en [/etc/shadow](#arch_shadow)
- UID: El identificador numerico del usuario
- GID: El identificador numerico del grupo principal del usuario
- descripcion: Una descripcion sobre el usuario
- dir_ini: Directorio de inicio del usuario
- shell: La terminal de inicio del usuario

## Seguridad del usuario <a id="user_security">

Este apartado se encuentra en el archivo *``/etc/shadow``* con la siguiente estructura:

~~~
<nombre>:<contraseña>:<ultimo_cambio>:<dias_min>:<dias_max>:<dias_aviso>:<dias_inactividad>:<dias_caducidad>:<reservado>
~~~

- nombre: El nombre del usuario
- contraseña: La contraseña encriptada del usuario ([estructura](#struct_password))
- ultimo_cambio: La fecha del último cambio de contraseña en formato de días desde el 1 de enero de 1970 (<a href="https://espanol.epochconverter.com/" target="_blank">pagina para calcularlo</a>)
- dias_min: El número mínimo de días que deben pasar antes de que se pueda cambiar la contraseña
- dias_max: El número máximo de días que la contraseña es válida antes de que deba ser cambiada
- dias_aviso: El número de días antes de que la contraseña expire que se mostrará un aviso
- dias_inactividad: El número de días que un usuario puede estar inactivo antes de que su cuenta se desactive
- dias_caducidad: La fecha de caducidad de la cuenta en formato de días desde el 1 de enero de 1970
- reservado: Campo reservado para uso futuro

Se puede obviar un campo no escribiendolo


## Añadir usuario <a id="add_user">

Para añadir un usuario de forma manual hay que seguir los siguientes pasos:

1. Crear el usuario en [*``/etc/passwd``*](#usuarios)
2. Añadir los datos de seguridad al usuario en [*``/etc/shadow``*](#user_security)
3. Añadir los grupos en [*``/etc/group``*](#grupos)
4. Copiar los ficheros de configuracion de *``/etc/skel/*``* en el directorio *``$HOME``* del nuevo usuario

Tambien se puede hacer de forma automatica con el comando ``adduser``

## Eliminar usuario <a id="del_user">

Se puede eliminar de varias maneras:

- Cambiando la shell a *``/bin/false``*, para bloquear temporalmente al usuario pero no es seguro ya que se pueden seguir conectando a traves de ftp
- Bloqueo de cuenta: se puede bloquear la cuenta cambiando la contraseña invalida, esto se puede hacer con el siguiente comando:

        passwd -l <usuario>

- Eliminar la cuenta: Se puede eliminar la cuenta con el siguiente comando:

        userdel [-r] <usuario> 

>    -r removes user files from /home/\<usuario>

## Cambiar contraseña a usuario <a id="change_password">

Para cambiar la contraseña se puede utilizar el comando ``passwd``

## Estructura de las contraseñas <a id="struct_password">

Estos son algunos ejemplos de es formato que tienen las contraseñas encriptadas con un hash, se separa por el simbolo ``$``, el primer valor es la funcion encriptacion y despues depende del formato, puede ser un salto para modificar la salida o el propio hash


| Tipo de encriptación | Formato                        | Ejemplo                                                                                   |
|----------------------|--------------------------------|-------------------------------------------------------------------------------------------|
| MD5                  | `$1$<salt>$<hash>`            | `$1$A0s7.D4$2sLgcmtCc0J5PbG/zCZbS.`                                                      |
| SHA-256              | `$5$<salt>$<hash>`            | `$5$Vf5x2I82$Ou3HC3vCBMsB4CnAvQk1hqXfZpW1SR5wGpzXkGGCrS0`                                |
| SHA-512              | `$6$<salt>$<hash>`            | `$6$J3bScLss$I20GqO9omVb2o8.D/SfZCuBYnUmRCbnu.l4Gfk1.p1tAMKqB9d6z0F.H8Ivgh1T6N8TlGbGf5JtQXhZqzF7pT0` |
| bcrypt               | `$2y$<cost>$<salt><hash>`     | `$2y$10$XhT.5p4ZyjA9bEMly6jJH.jVRkLb2U0w43F.FT9A4JBYqsLr5n.yG`                           |

## Permisos de ficheros y archivos <a id="perm_fich_arch">

En un fichero o archivo se pueden dar permiso a 3 grupos, el propietario, el grupo y el resto de usuarios

### Ver permisos <a id="see_perm">

Para ver los permisos se utiliza el comando ``ls -l`` y la salida va a ser algo como lo siguiente:

~~~
drwxr-xr-x 2 usuario grupo 4096 mar 24 10:00 carpeta_ejemplo
~~~

- El primer caracter es el tipo de archivo (d para directorio y - para archivo comun)
- El resto son los permisos de 3 en 3: usuario, gurpo y el resto respectivamente.
    - r: permiso de lectura
    - w: permiso de escritura
    - x: permiso de ejecucion
- El siguiente numero es el numero de enlaces, el numero de punteros que apuntan al inodo del archivo o fichero
- El siguiente es el tamaño del archivo en bytes
- La siguiente cadena es la ultima fecha de modificacion
- Lo ultimo es el nombre del archivo o directorio

### Cambiar permisos <a id="change_perm">

Lo priemro es poder cambiar propietario y el grupo al que se le dan los permisos, para esto se puede utilizar el siguiete comando:

~~~
chown <propietario>:<grupo> <archivo>
~~~

Se pueden ignorar el propietario o el grupo para solo cambiar uno de ellos pero hay que poner ``:``

Lo siguiente es poder cambiar los permisos, se hace con el siguiente comando:

~~~
chmod [permisos] <archivo>
~~~

Los permisos se escriben de la siguiente forma:

- ``+``: para añadir un permiso
- ``-``: para quietar un permiso
- ``=``: para fijarlos permisos un permiso

Se selecciona a quien se le dan los permisos de la siguiente forma:

- ``u``: Indica el usuario (propietario) del archivo.
- ``g``: Indica el grupo del archivo.
- ``o``: Indica otros usuarios (usuarios que no son el propietario ni están en el grupo del archivo).
- ``a``: Indica todos los usuarios

Para seleccionar varios de estos en el mismo comando se separan con ``,``

Por ultimo, los permisos:

- ``r``: permiso de lectura
- ``w``: permiso de escritura
- ``x``: permiso de ejecucion

## SUDO <a id="sudo">

Los permisos de sudo se editan en el archivo *``/etc/sudoers``*, para editar este archivo se recomienda usar ``visudo`` ya que verifica la sintaxis para evitar errores.

La estructura es la siguiente:

~~~
<usuario/grupo> <hosts>=(<usuario_convertido>) <comando>
~~~

- usuario/grupo: El usuario/grupo que puede ejecutar el comando, si es un grupo hay que escribir ``%`` antes (%grupo)
- host: Desde que hosts se puede ejecutar (poner ALL)
- usuario_converido: En que usuario se convierte para ejecutarlo (poner ALL)
- comando: El comando que quieres dar los permisos, si quieres poner varios separarlos por ``,``. Es util el comando [*``which``*](#comd_interes).

## Comandos de interes <a id="comd_interes">

- *``which [comando]``*: Sirve para ver donde esta situado un comando
