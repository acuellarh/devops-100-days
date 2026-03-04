# Day 4: Script Execution Permissions

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

In a bid to automate backup processes, the xFusionCorp Industries sysadmin team has developed a new bash script named xfusioncorp.sh. While the script has been distributed to all necessary servers, it lacks executable permissions on App Server 1 within the Stratos Datacenter.

Your task is to grant executable permissions to the /tmp/xfusioncorp.sh script on App Server 1. Additionally, ensure that all users have the capability to execute it.

_Conectarse mediante SSH al server_  

```
ssh usuario@ip_o_serverName
ssh tony@172.16.238.10
```
_Una vez conectado, se digita el correspondiente password. Una vez dentro del server se verificar los permisos actuales del archivo

```
[tony@stapp01 tmp]$ ls -la | grep *.sh
---------- 1 root root   40 Mar  4 01:35 xfusioncorp.sh
```

Se requiere habilitar los permisos de ejecución del archivo, sin embargo para ejecutar tambien se requiere leer. Adicional al dueño del archivo le vamos asignar permisos de escritura:

Lo correcto es

```
rwx = 4+2+1 = 7
r-1 = 4+0+1 = 5
r-1 = 4+0+1 = 5

```

```
[tony@stapp01 tmp]$ sudo chmod 755 /tmp/xfusioncorp.sh 

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony: 
[tony@stapp01 tmp]$ ls -la | grep *.sh
-rwxr-xr-x 1 root root   40 Mar  4 02:05 xfusioncorp.sh

```

Se procede a verificar que todos los usuarios puede ejecutar y leer, adicional que el dueño puede escribir.
