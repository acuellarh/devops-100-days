# Day 6: Create a Cron Job

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

The Nautilus system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in Stratos DC on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:

a. Install cronie package on all Nautilus app servers and start crond service.

b. Add a cron */5 * * * * echo hello > /tmp/cron_text for root user.

_1. Conectarse mediante SSH al server_  

Para este caso se realiza el comando con el nombre corto del host dado que así no genera problema

```
ssh usuario@ip_o_serverName
ssh tonystapp01
```
Una vez conectado, se digita el correspondiente password. 

## Contexto cronie

cronie es un paquete programador de tareas para Linux. Proporciona el demonio crond (un servicio en segundo plano) que lee los cron jobs — comandos programados — y los ejecuta automáticamente en los momentos especificados.

Piensa en él como poner una alarma, pero para comandos de Linux.

¿Qué es un cron job?
Un cron job es un comando programado para ejecutarse repetidamente en un intervalo de tiempo definido. El horario se define usando una sintaxis especial llamada expresión cron:

```
* * * * *  comando
│ │ │ │ │
│ │ │ │ └── Día de la semana (0-7, Dom=0 o 7)
│ │ │ └──── Mes (1-12)
│ │ └────── Día del mes (1-31)
│ └──────── Hora (0-23)
└────────── Minuto (0-59)

Entonces */5 * * * * significa: "cada 5 minutos, cada hora, cada día"


```
1. Para ejecutar los comandos como root, ejecutamos el siguiente comando

```
[tony@stapp01 ~]$ sudo su -

[sudo] password for tony: 
[root@stapp01 ~]# 
```

Posterior, instalamos el paquete 'cronie'

```
[root@stapp01 ~]# yum install -y cronie
```

La salida esperada es 


```
Installed: cronie.x86_64
Complete!
```

2. Iniciar y habilitar el servicio 'crond'

```
systemctl start crond

systemctl enable crond

```
systemctl es la herramienta para administrar servicios de Linux services (start, stop, restart, enable/disable). crond  es el actual 'daemon process' que correo los 'cron jobs'.

Para verificar el estado, utilizamos el comando

```
systemctl status crond

You should see active (running) in green. ✅

Active: active (running) since Mon 2026-03-09 20:21:27 UTC; 3min 20s ago

```

3. Adicionar el cron job 

```
crontab -e

```
Explicación del comando

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

4. Verificar que el cron job fue guardado


```
crontab -l

```
Explicación

```

> `Lista todos los 'cron jobs' para el usuario en el que estamos logueados, para este caso el root.

Output should show:

*/5 * * * * echo hello > /tmp/cron_text
```

5. Verificación final, luego de 5 minutos


```
cat /tmp/cron_text
```

Le debe aparecer el texto 'hello'
