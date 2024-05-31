## 1

### a)

Procesos : ``ps aux``

### b)

Obtener los tamaños: ``df -h```

### c) 

``free``

## 2

El comando ``systemd-cat`` en sistemas Linux se utiliza para enviar mensajes a los logs del sistema gestionados por systemd. Se utiliza pasnadolo por una pipe 

## 3

Para cambiarlo se hace desde el siguiente archivo ``/etc/systemd/journald.conf``

Descomentar y cambiar la siguiente linea ``ForwardToConsole=INFO``

## 4

Comando ``journalctl`` para ver todos los logs

## 5

``journalctl -k --since "1 hour ago"``

## 6

Descomentar LogLevel y cambiar a DEBUG

Reiniar el ssh con ``systemctl restart sshd``

## 7