# Day 1: Linux User Setup with Non-Interactive Shell

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

Crear un usuario en una servidor con shell no interactivo, dado que se requiere crear el usuario para 
poder realizar un proceso de backup

_Conectarse mediante SSH al server_  

```
ssh usuario@ip_o_serverName
ssh tony@172.16.238.10
```
_Una vez conectado, se digita el correspondiente password_, posteriormente se procede con el comando para crear el usuario con el shell no interactivo

```
sudo useradd -s /sbin/nologin nombre_usuario
```
Se utiliza este comando para donde se requiere que el usuario exista, pero no haga login.
Para verificar la correcta creación.

```
grep "texto_patron" ruta_nombreArchivo
grep jhon /etc/passwd
```
Tambien

```
hostname
geten
