# Day 7: Linux SSH Authentication

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

The system admins team of xFusionCorp Industries has set up some scripts on jump host that run on regular intervals and perform operations on all app servers in Stratos Datacenter. To make these scripts work properly we need to make sure the thor user on jump host has password-less SSH access to all app servers through their respective sudo users (i.e tony for app server 1). Based on the requirements, perform the following:
Set up a password-less authentication from user thor on jump host to all app servers through their respective sudo users.

## Contexto Autenticación SSH sin contraseña

Funciona como un candado y una llave:

```
Jump Host (thor)                    App Servers
┌─────────────────────┐             ┌─────────────────┐
│                     │             │                 │
│  🔑 Llave Privada   │────SSH─────▶│ 🔒 Llave Pública│
│  (id_rsa)           │             │ (authorized_keys)│
│  ← NUNCA sale       │             │ ← se copia aquí │
│    de aquí          │             │                 │
└─────────────────────┘             └─────────────────┘


Llave privada → se queda en tu máquina, nunca la compartes
Llave pública → la copias a los servidores a los que quieres acceder
Cuando haces SSH, el servidor verifica que tu llave privada corresponde con la pública → te deja entrar sin pedir contraseña
```



**Paso 1. Verificar si thor ya tiene llaves SSH**  

```
thor@jump-host ~$ ls ~/.ssh/
ls: cannot access '/home/thor/.ssh/': No such file or directory
```

Si ves _id_rsa_ y _id_rsa.pub_ ya tienes llaves generadas, salta al Paso 3.

**Paso 2.  Generar el par de llaves SSH**

```
ssh-keygen -t rsa

```


Te hará 3 preguntas, **presiona Enter en todas** para dejar valores por defecto:

```
Enter file in which to save the key (/home/thor/.ssh/id_rsa):  ← Enter
Enter passphrase (empty for no passphrase):                     ← Enter
Enter same passphrase again:                                     ← Enter

-t rsa → tipo de encriptación (RSA es el estándar)
Sin passphrase → así será verdaderamente sin contraseña
```
Verifica que se crearon:

```
thor@jump-host ~$ ls ~/.ssh/
id_rsa  id_rsa.pub
thor@jump-host ~$ 

# Debes ver:
# id_rsa       ← llave privada (nunca compartas esto)
# id_rsa.pub   ← llave pública (esta se copia a los servidores)
```

** Paso 3 — Copiar la llave pública a cada servidor **

El comando `ssh-copy-id` hace todo automáticamente — copia tu llave pública al archivo ~/.ssh/authorized_keys del servidor destino.

```
# App Server 1 - tony
ssh-copy-id tony@stapp01

# App Server 2 - steve
ssh-copy-id steve@stapp02

# App Server 3 - banner
ssh-copy-id banner@stapp03
```

Para cada uno te pedirá la contraseña **una última vez**:
```
tony@stapp01's password: Ir0nM@n
steve@stapp02's password: Am3ric@
banner@stapp03's password: BigGr33n

```
Esta es la **única vez** que escribes la contraseña. Después de esto, nunca más. 

Se relaciona a continuación el ejemplo de las salida para el el server stapp01

```
thor@jump-host ~$ ssh-copy-id tony@stapp01
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/thor/.ssh/id_rsa.pub"
The authenticity of host 'stapp01 (10.244.73.153)' can't be established.
ED25519 key fingerprint is SHA256:N2W40VBAqPYWidHBtkouMUXO4e7+mqyn97fHjjzBtlE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
tony@stapp01's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'tony@stapp01'"
and check to make sure that only the key(s) you wanted were added.

thor@jump-host ~$ ssh tony@stapp01
[tony@stapp01 ~]$ 

```

Al final se evidencia que se puede establecer la conexión sin digitar la contraseña.

**Paso 4 — Verificar con un comando directo (sin entrar al servidor)**

```
# Ejecutar comando remoto sin entrar al servidor
ssh tony@stapp01 "hostname && whoami"
ssh steve@stapp02 "hostname && whoami"
ssh banner@stapp03 "hostname && whoami"
```

Debes ver la respuesta inmediata sin pedir contraseña:
```
stapp01
tony

```
¿Qué hizo `ssh-copy-id` por dentro?
Cuando se ejecuta  `ssh-copy-id tony@stapp01`, esto es lo que pasó:

```

> **`crontab -e`** abre 

 contrab -> Abre la tabla de los cron (schedule file) para el **current user** (root, ya que nos cambiamos al root ) in a text editor.
 
 `-e` = edit -> Esto abre el editor `vi`/`vim`. A continuación como utilizarlo:

 1. Presione  i       → Ingresa al modo de inserción
 2. Digita la línea del cron job:
   */5 * * * * echo hello > /tmp/cron_text
 3. Presione  Esc     → exits INSERT mode
 4. Digite  :wq      → saves and quits (:w = write, :q = quit)
 5. Presione  Enter

 Le debe aparecer un mensaje: 
 crontab: installing new crontab

```


_ssh-copy-id básicamente hizo esto automáticamente:_

```
cat ~/.ssh/id_rsa.pub | ssh tony@stapp01 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

```
Tu llave pública del server jump quedó guardada en cada uno de los servidores configurados:

_En stapp01, dentro de /home/tony/.ssh/authorized_keys:_
```
ssh-rsa AAAAB3NzaC1yc2EAAA...(tu llave pública)... thor@jumphost
```

Cada vez que te conectas, SSH verifica automáticamente que tu llave privada corresponde → acceso garantizado sin contraseña.
