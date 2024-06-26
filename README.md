# <img src="https://cdn.icon-icons.com/icons2/195/PNG/256/OS_Linux_23399.png" width="50"> Apuntes Sistemas Informaticos

### Conectarte por SSH

~~~bash
ssh -p 2222 root@127.0.0.1
~~~

### Compartir archivos

**Conectarse**

~~~bash
sftp -P 2222 root@127.0.0.1
~~~

put/get \<origen> \<destino>: para enviar/recibir archivo

### Indice

- [Practica 2](Practica2.md)
    - [Añadir un disco a la maquina virtual](Practica2.md#add_disc_maq)
    - [Inodos](Practica2.md#inodos)
    - [Crear archivos random](Practica2.md#crear_arch_random)
    - [Particiones](Practica2.md#particones)
        - [Hacer una particion del disco](Practica2.md#hacer_part)
        - [Tabla tamaño de particiones](Practica2.md#tabla_tam_part)
        - [Fijar sistema de ficheros](Practica2.md#fijar_sis_ficheros)
        - [Montar la particion](Practica2.md#montar_particion)
        - [Fichero de montado de particiones](Practica2.md#fich_mont_particiones)
        - [Desmontar un disco](Practica2.md#des_disc)
        - [Juntar particiones](Practica2.md#juntar_part)
        - [Redimensionar particiones](Practica2.md#redimenisonar_part)
        - [Comandos de interres](Practica2.md#comd_interes)
    - [Encriptacion y desencriptacion](Practica2.md#encriptacion_desencriptacion)
        - [Encriptacion de archivo](Practica2.md#encriptacion_arch)
        - [Desencriptacion de archivo](Practica2.md#desencriptacion_arch)
        - [Encriptacion de particion](Practica2.md#encriptacion_part)
- [Practica 3](Practica3.md)
    - [Grupos](Practica3.md#grupos)
    - [Usuarios](Practica3.md#usuarios)
    - [Seguridad del usuario](Practica3.md#user_security)
    - [Añadir usuario](Practica3.md#add_user)
    - [Eliminar usuario](Practica3.md#del_user)
    - [Cambiar contraseña a usuario](Practica3.md#change_password)
    - [Estructura de las contraseñas](Practica3.md#struct_password)
    - [Permisos de ficheros y archivos](Practica3.md#perm_fich_arch)
        - [Ver permisos](Practica3.md#see_perm)
        - [Cambiar permisos](Practica3.md#change_perm)
    - [SUDO](Practica3.md#sudo)
    - [Comandos de interes](Practica3.md#comd_interes)
- [Practica 4](Practica4.md)
    - [Instalacion desde codigo fuente](Practica4.md#rep_inst)
    - [Instalacion desde paquete](Practica4.md#inst_pack)
        - [Instalar un paquete](Practica4.md#apt_install)
        - [Buscar un paquete](Practica4.md#apt_search)
        - [Refrescar lista de paquetes](Practica4.md#apt_update)
        - [Actualizar los paquetes](Practica4.md#apt_upgrade)
        - [Añadir repositorios](Practica4.md#apt_add_repository)
- [Practica 5](Practica5.md)
    - [Interfaces](Practica5.md#interfaces)
    - [Configuracíon de las interfaces](Practica5.md#conf_interfaces)
    - [ARP](Practica5.md#arp)
    - [Tablas de rutas](Practica5.md#tablas_de_rutas)
    - [Resolucion de nombres](Practica5.md#resol_nombres)
    - [Archivo de configuracion de DNS](Practica5.md#arch_conf_dns)
    - [Utilizar el ordenador como router (ruteador)](Practica5.md#routeador)
    - [Comandos utiles](Practica5.md#comandos_utiles)
- [Practica 6](Practica6.md)
    - [Fases del arranque](Practica6.md#fases_arranque)
    - [UEFI](Practica6.md#UEFI)
        - [UEFI Shell](Practica6.md#UEFI_shell)
            - [**Error en UEFI Shell**](Practica6.md#UEFI_shell_error)
            - [**Disco de arranque**](Practica6.md#disco_arranque)
        - [Orden de arranque](Practica6.md#ord_arranque)
        - [Gestor de arranque](Practica6.md#gest_arranque)
    - [GRUB](Practica6.md#grub)
        - [Configuracion](Practica6.md#grub_conf)
            - [Archivo grub ](Practica6.md#arch_grub)
            - [Directorio grub.d](Practica6.md#dir_grub.d)
        - [Linea de comandos de GRUB](Practica6.md#grub_cmd)
    - [Kernel](Practica6.md#kernel)
            - [Init Ramdisk](Practica6.md#ramdisk)
    - [Servicios](Practica6.md#servicios)
        - [Comandos de interes](Practica6.md#cmd_servicios)
            - [Ver los servicios:](Practica6.md#cmd_servicios_ver_servicios)
            - [Iniciar/Detener un servicio:](Practica6.md#cmd_servicios_iniciar_detener_servicio)
            - [Reiniciar un servicio ](Practica6.md#cmd_servicios_reiniciar_servicio)
            - [Recargar la informacion de un servicio](Practica6.md#cmd_servicios_recargar_servicio)
            - [Habilitar/Deshabilitar un servicio para que arranque automaticamente ](Practica6.md#cmd_servicios_habilitar_deshabilitar_servicio)
            - [Mostrar las dependencias de un servicio](Practica6.md#cmd_servicios_dependencias)
            - [Recargar los servicios ](Practica6.md#cmd_servicios_daemon_reload)
        - [Implementacion de servicios](Practica6.md#impl_servicios)
            - [\[Unit\]](Practica6.md#impl_servicios_unit)
            - [\[Service\]](Practica6.md#impl_servicios_service)
            - [\[Install\]](Practica6.md#impl_servicios_install)
            - [Timers](Practica6.md#servicios_timer)
- [Practica 7](Practica7.md)
    - [LVM (Logical Volume Manager)](Practica7.md#lvm)
        - [Administador de LVM (lvm2)](Practica7.md#administrador_lvm)
    - [RAID](Practica7.md#raid)
        - [Niveles](Practica7.md#raid_lev)
        - [Acciones](Practica7.md#raid_acc)
        - [Crear una raid](Practica7.md#raid_create)
        - [Recuperar un disco](Practica7.md#raid_recuperate)
    - [Backups](Practica7.md#backups)
        - [Realizar backup](Practica7.md#blackup_make)
        - [Restaurar backup](Practica7.md#blackup_restore)
- [Practica 8](Practica8.md)
    - [Monitorizacion de logs](Practica8.md#logs_monitor)
    - [Gestion de procesos](Practica8.md#gest_proc)
        - [Monitorizar procesos](Practica8.md#mon_proc)
        - [Logs de procesos](Practica8.md#logs_proc)
        - [Detener procesos](Practica8.md#kil_proc)
        - [Prioridad de los procesos](Practica8.md#proc_prio)
        - [Limitacion de recursos](Practica8.md#limt_rec)
    - [Gestion de memoria](Practica8.md#gest_mem)
        - [Limintar memoria](Practica8.md#lim_mem)
            - [Ver las cuotas](Practica8.md#lim_mem_ver)
            - [Preparar el sistema](Practica8.md#lim_mem_prep)
            - [Editar las cuotas](Practica8.md#lim_mem_edit)
        - [Memoria Swap](Practica8.md#men_swap)
            - [Memoria Swap en particion](Practica8.md#mem_swap_pat)
            - [Memoria Swap en archivo](Practica8.md#mem_swap_arch)
            - [Eliminar memoria Swap](Practica8.md#mem_swap_del)
    - [Comandos utiles](Practica8.md#utl_cmd)

<img src="https://cdn.nubika.es/wp-content/uploads/2021/05/caracteristicas-del-orangutan.jpg">
