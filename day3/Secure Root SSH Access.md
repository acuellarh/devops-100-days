# Day 3: Secure Root SSH Access

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

Following security audits, the xFusionCorp Industries security team has rolled out new protocols, including the restriction of direct root SSH login.

Your task is to disable direct SSH root login on all app servers within the Stratos Datacenter.

_Conectarse mediante SSH al server_  

```
ssh usuario@ip_o_serverName
ssh steve@172.16.238.11
```
_Una vez conectado, se digita el correspondiente password_, posteriormente se procede con el comando para crear el usuario 'james' con la fecha de expiración

```
sudo grep PermitRootLogin /etc/ssh/sshd_config
```

la respuesta fue 

```
PermitRootLogin yes
```

Se procede a verificar si otros usuarios tienen el permiso de root o sudo

```
awk -F: '$3 >= 1000 {print $1}' /etc/passwd | xargs -I{} bash -c 'echo -n "{}: "; groups {}'
```
la respuesta fue 
```
nobody: nobody : nobody
ansible: ansible : ansible wheel
tony: tony : tony wheel
[tony@stapp01 ~]$ 
```
Tanto ansible como tony están en el grupo wheel — en esta distro (CentOS/RHEL) el grupo wheel equivale a sudo.
Entonces tony SÍ tiene permisos de root

Ahora se procede con el comando para realizar el ajuste y no permitir el login del usuario root

```
sudo sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

Verificar el cambio

```
sudo grep PermitRootLogin /etc/ssh/sshd_config
```

Debe mostrar

```
PermitRootLogin no
```
