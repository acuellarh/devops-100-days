# Day 14: Linux Process Troubleshooting

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC.

Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not have hosted any code yet on these servers, so you don't need to worry if Apache isn't serving any pages. Just make sure the service is up and running. Also, make sure Apache is running on port 3004 on all app servers.



**A continuación se relacionan las actividades a realizar en cada server, dependiendo del resultado de los comandos de diagnostico**  


```
ssh tony@stapp01 -y

```
Verificar el estado del servicio de apache para stapp01:

```
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Mon 2026-04-06 22:50:06 UTC; 7min ago
       Docs: man:httpd.service(8)
    Process: 22979 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 22979 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 36ms

Apr 06 22:50:05 stapp01 systemd[1]: Starting The Apache HTTP Server...
Apr 06 22:50:06 stapp01 httpd[22979]: AH00558: httpd: Could not reliably determine the server's fully quali>
Apr 06 22:50:06 stapp01 httpd[22979]: (98)Address already in use: AH00072: make_sock: could not bind to add>
Apr 06 22:50:06 stapp01 httpd[22979]: (98)Address already in use: AH00072: make_sock: could not bind to add>
Apr 06 22:50:06 stapp01 httpd[22979]: no listening sockets available, shutting down
Apr 06 22:50:06 stapp01 httpd[22979]: AH00015: Unable to open logs
Apr 06 22:50:06 stapp01 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Apr 06 22:50:06 stapp01 systemd[1]: httpd.service: Failed with result 'exit-code'.
Apr 06 22:50:06 stapp01 systemd[1]: Failed to start The Apache HTTP Server.
lines 1-18/18 (END)

```
Se identifica que el servicio no se encuentra activo, se intenta iniciar el servicio 

```
[tony@stapp01 ~]$ systemctl start httpd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to start 'httpd.service'.
Authenticating as: tony
Password: 
==== AUTHENTICATION COMPLETE ====
Job for httpd.service failed because the control process exited with error code.
See "systemctl status httpd.service" and "journalctl -xeu httpd.service" for details.
```

Se verifica el detalle del estado del servicio

```
[tony@stapp01 ~]$ systemctl status httpd.service
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Mon 2026-04-06 22:59:06 UTC; 40s ago
       Docs: man:httpd.service(8)
    Process: 48465 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 48465 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 31ms

Apr 06 22:59:06 stapp01 systemd[1]: Starting The Apache HTTP Server...
Apr 06 22:59:06 stapp01 httpd[48465]: AH00558: httpd: Could not reliably determine the server's fully quali>
Apr 06 22:59:06 stapp01 httpd[48465]: (98)Address already in use: AH00072: make_sock: could not bind to add>
Apr 06 22:59:06 stapp01 httpd[48465]: (98)Address already in use: AH00072: make_sock: could not bind to add>
Apr 06 22:59:06 stapp01 httpd[48465]: no listening sockets available, shutting down
Apr 06 22:59:06 stapp01 httpd[48465]: AH00015: Unable to open logs
Apr 06 22:59:06 stapp01 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Apr 06 22:59:06 stapp01 systemd[1]: httpd.service: Failed with result 'exit-code'.
Apr 06 22:59:06 stapp01 systemd[1]: Failed to start The Apache HTTP Server.
lines 1-18/18 (END)
```
En la salida se encuentra que el puerto ya se encuentra utilizado por otro programa, se procede a realizar la validación.

Con sudo para ver el PID del proceso que ocupa el puerto. Luego:

```
[tony@stapp01 ~]$ sudo ss -tlnp | grep 3004

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony: 
LISTEN 0      10         127.0.0.1:3004      0.0.0.0:*    users:(("sendmail",pid=22305,fd=4))
[tony@stapp01 ~]$ 
```

kill -9 PIDSIGKILL Fuerza la terminación inmediata. El proceso no puede ignorarla

```
[tony@stapp01 ~]$ sudo kill -9 22305
```

Se procede a iniciar y habilitar el servicio

```
[tony@stapp01 ~]$ systemctl start httpd
[tony@stapp01 ~]$ systemctl enable httpd
```

Se verifica nuevamente el servicio de apache y que servicio esta corriendo bajo el puerto 3004

```
[tony@stapp01 ~]$ systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: disabled)
     Active: active (running) since Mon 2026-04-06 23:10:32 UTC; 1min 19s ago
       Docs: man:httpd.service(8)
   Main PID: 51464 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 177 (limit: 404712)
     Memory: 15.0M
        CPU: 99ms
     CGroup: /system.slice/httpd.service
             ├─51464 /usr/sbin/httpd -DFOREGROUND
             ├─51471 /usr/sbin/httpd -DFOREGROUND
             ├─51472 /usr/sbin/httpd -DFOREGROUND
             ├─51473 /usr/sbin/httpd -DFOREGROUND
             └─51474 /usr/sbin/httpd -DFOREGROUND

Apr 06 23:10:32 stapp01 systemd[1]: Starting The Apache HTTP Server...
Apr 06 23:10:32 stapp01 httpd[51464]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.73.162>
Apr 06 23:10:32 stapp01 httpd[51464]: Server configured, listening on: port 3004
Apr 06 23:10:32 stapp01 systemd[1]: Started The Apache HTTP Server.
lines 1-20/20 (END)

[tony@stapp01 ~]$ sudo ss -tlnp | grep 3004
[sudo] password for tony: 
LISTEN 0      511                *:3004            *:*    users:(("httpd",pid=51474,fd=4),("httpd",pid=51473,fd=4),("httpd",pid=51472,fd=4),("httpd",pid=51464,fd=4))
[tony@stapp01 ~]$ 

```

En resumen, se verifica el estado del servicio, en caso que otro servicio ya ocupe el puerto se procede a finalizar el proceso, se inicia el servicio de apache en el puerto establecido y como buena practiva se habilita el servicio para que este inicie nuevamente en caso de reinicio del server.

