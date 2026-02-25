# Day 2: Day 2: Temporary User Setup with Expiry

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

As part of the temporary assignment to the Nautilus project, a developer named james requires access for a limited duration. To ensure smooth access management, a temporary user account with an expiry date is needed. Here's what you need to do:



Create a user named james on App Server 2 in Stratos Datacenter. Set the expiry date to 2026-12-07, ensuring the user is created in lowercase as per standard protocol.

_Conectarse mediante SSH al server_  

```
ssh usuario@ip_o_serverName
ssh steve@172.16.238.11
```
_Una vez conectado, se digita el correspondiente password_, posteriormente se procede con el comando para crear el usuario 'james' con la fecha de expiración

```
sudo useradd -e 2026-12-07 james
```
Tambien le puedes asignar una contraseña

```
sudo passwd james
Changing password for user james.
New password: 
Retype new password:
passwd: password updated successfully
```

Para verificar que fue agregado correctamente

```
[steve@stapp02 ~]$ sudo chage -l james
Last password change                                    : Feb 24, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Dec 07, 2026
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
[steve@stapp02 ~]$ 
```

Le asigno una contraseña al usuario james

```
sudo passwd james
Changing password for user james.
New password: 
Retype new password:
passwd: password updated successfully
```

Finalizo la conexión con el usuario steve(admon) y pruebo la conexión con el usuario james

```
[steve@stapp02 ~]$ exit
logout
Connection to 172.16.238.11 closed.
thor@jumphost ~$ ssh james@172.16.238.11
james@172.16.238.11's password: 
Last failed login: Tue Feb 24 22:33:59 UTC 2026 from 172.16.238.3 on ssh:notty
There was 1 failed login attempt since the last successful login.
[james@stapp02 ~]$ whoami
james
[james@stapp02 ~]$ 
```
