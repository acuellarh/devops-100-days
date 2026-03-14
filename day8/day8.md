# Day 8: Install Ansible

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

During the weekly meeting, the Nautilus DevOps team discussed about the automation and configuration management solutions that they want to implement. While considering several options, the team has decided to go with Ansible for now due to its simple setup and minimal pre-requisites. The team wanted to start testing using Ansible, so they have decided to use jump host as an Ansible controller to test different kind of tasks on rest of the servers.

Install ansible version 4.7.0 on Jump host using pip3 only. Make sure Ansible binary is available globally on this system, i.e all users on this system are able to run Ansible commands.

## Contexto Autenticación SSH sin contraseña

* Jump Host: servidor "puente" desde donde se controlan otros servidores. En DevOps es común usarlo como punto central de administración.
* Ansible**: herramienta de automatización que permite configurar servidores remotamente sin instalar agentes en ellos (solo necesita SSH).
* pip3: gestor de paquetes de Python 3.
* "disponible globalmente": que el binario de ansible esté en una ruta del sistema (/usr/bin/ o similar), no solo para tu usuario.


**Paso 1. Instalar Ansible con pip3 de forma global**  

La clave está en usar `sudo` con `pip3` para instalar en las rutas del sistema:

```
sudo pip3 install ansible==4.7.0
```

Esto instala ansible en `/usr/local/bin/ansible`, que es accesible para todos los usuarios.

**Paso 2.  Verificar que quedó instalado correctamente**

```
ansible --version
```

Deberías ver algo como:
```
ansible [core 2.11.x]
...

```

**Paso 3 — Verificar que el binario es global**

```
which ansible
```
Debe mostrar `/usr/local/bin/ansible` (ruta del sistema, no de usuario).

---

¿Por qué `sudo pip3` y no solo `pip3`?

| Comando | Instala en | Accesible por |
|---|---|---|
| `pip3 install ansible` | `~/.local/bin/` | Solo tu usuario |
| `sudo pip3 install ansible` | `/usr/local/bin/` | **Todos los usuarios**  |

---

## Resumen del flujo
```
Jump Host (Ansible Controller)
        │
        ├─── SSH → Servidor 1
        ├─── SSH → Servidor 2
        └─── SSH → Servidor N
```


Ansible no necesita nada instalado en los servidores destino, solo SSH activo
