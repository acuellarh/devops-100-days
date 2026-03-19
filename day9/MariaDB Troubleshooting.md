# Day 9: MariaDB Troubleshooting

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

There is a critical issue going on with the Nautilus application in Stratos DC. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that mariadb service is down on the database server.

Look into the issue and fix the same.

## Contexto MariaDB


**Paso 1. Conectarse al servido de BD y verifica rel estado del servicio de MariaDB**  


```
ssh peter@stdb01

sudo systemctl status mariadb
```
El resultado es el siguiente:

```
○ mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/

```
_¿Qué significaban esos dos campos?_

* **Loaded:** disabled → El servicio existe pero no está configurado para iniciar al arrancar el servidor. Si el servidor se reinicia, MariaDB no levanta solo.
* **Active:** inactive (dead) → El proceso simplemente no está corriendo en este momento.

**Paso 1. Iniciar el servicio**

Comando

```
sudo systemctl start mariadb
Job for mariadb.service failed because the control process exited with error code.
See "systemctl status mariadb.service" and "journalctl -xeu mariadb.service" for details.
...

```

El comando generó error, por lo cual es necesario identificar la razón, para lo cual se debe consultar el log de MariaDB

Comando

```
sudo cat /var/log/mariadb/mariadb.log

```

Mensaje de salida
```
... resto de código

2026-03-17  2:33:31 0 [ERROR] mariadbd: Can't create/write to file '/run/mariadb/mariadb.pid' (Errcode: 13 "Permission denied")
2026-03-17  2:33:31 0 [ERROR] Can't start server: can't create PID file: Permission denied
---

```
El directorio `/run/mariadb/` tiene permisos incorrectos — MariaDB no puede escribir su archivo PID ahí. Vamos a verificar los permisos y propiedad actual.

```
ls -ld /run/mariadb/
drwxr-xr-x 2 root mysql 40 Mar 16 22:37 /run/mariadb/
```
Ahí está el problema. El owner es `root` en lugar de `mysql`. El proceso de MariaDB corre como usuario `mysql` y necesita escribir en ese directorio, pero `root` es el dueño.

```
sudo chown mysql:mysql /run/mariadb/
```

A continuación se procede a iniciar, habilitar y verificar el estado del servicio

```
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
```

Resultado

```
● mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: disabled)
     Active: active (running) since Tue 2026-03-17 02:39:11 UTC; 1min 7s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 63847 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 8 (limit: 404712)
     Memory: 58.1M
        CPU: 173ms
     CGroup: /system.slice/mariadb.service
             └─63847 /usr/libexec/mariadbd --basedir=/usr
```

Se evidencia que el servicio cambio a -enabled- y a _active (running)_
