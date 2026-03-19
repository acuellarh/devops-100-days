# Day 5: SElinux Installation and Configuration

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for App server 3 in the Stratos Datacenter:

Install the required SELinux packages.

Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.

No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.

Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled.

_1. Conectarse mediante SSH al server_  

```
ssh usuario@ip_o_serverName
ssh banner@172.16.238.12
```
Una vez conectado, se digita el correspondiente password. 

## Contexto SELinux


SELinux (Security-Enhanced Linux) es un módulo de seguridad del kernel de Linux desarrollado originalmente por la NSA. Actúa como una capa adicional de control de acceso obligatorio (MAC - Mandatory Access Control) sobre los permisos tradicionales de Linux.
En términos simples: aunque un proceso tenga permisos de usuario para acceder a un archivo, SELinux puede bloquearlo igualmente si su política de seguridad no lo permite. Es especialmente usado en servidores de producción (Red Hat, CentOS, Fedora).
Tiene 3 modos:

```
Enforcing → activo y bloqueando
Permissive → activo pero solo registra (no bloquea)
Disabled → completamente apagado
```

_1.Realizar la instalación._ 

```
[banner@stapp03 ~]$ sudo yum install -y selinux-policy selinux-policy-targeted policycoreutils
```

Una vez finaliza la instalación de manera exitosa 

```
....codigo previo

Installed:
  diffutils-3.7-12.el9.x86_64                      libselinux-utils-3.6-3.el9.x86_64                        
  policycoreutils-3.6-5.el9.x86_64                 rpm-plugin-selinux-4.16.1.3-38.el9.x86_64                
  selinux-policy-38.1.74-1.el9.noarch              selinux-policy-targeted-38.1.74-1.el9.noarch             

Complete!
[banner@stapp03 ~]$ 

```
_2. Deshabilitar SELinux permanentemente_

Edita el archivo de configuración:

```
vi /etc/selinux/config

```

Este es el contenido original:

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/changing-selinux-states-and-modes_using-selinux#changing-selinux-modes-at-boot-time_changing-selinux-states-and-modes
#
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

```

Busca la línea 

```
`SELINUX=enforcing` (o `permissive`) y cámbiala a:

SELINUX=disabled

```

La versión final queda de la siguiente manera

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/changing-selinux-states-and-modes_using-selinux#changing-selinux-modes-at-boot-time_changing-selinux-states-and-modes
#
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

```
