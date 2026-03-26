# Day 12: Linux Network Services

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

Our monitoring tool has reported an issue in Stratos Datacenter. One of our app servers has an issue, as its Apache service is not reachable on port 8082 (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue.



Use tools like telnet, netstat, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings.

Once fixed, you can test the same using command curl http://stapp02:8082 command from jump host.

Note: Please do not try to alter the existing index.html code, as it will lead to task failure.

## Contexto

Comandos 

```
ss (socket statistics)

-tlnp

| Opción | Significado |
|---|---|
| `-t` | Muestra solo conexiones **TCP** |
| `-l` | Muestra solo puertos en modo **escucha** (listening) |
| `-n` | Muestra **números** de puerto en vez de nombres (ej: `6300` en vez de `http`) |
| `-p` | Muestra el **proceso/programa** que usa ese puerto |

```


## 🧠 ¿Qué es `iptables`?

`iptables` = "IP tables" = Tablas de reglas para filtrar tráfico IP

Es el **firewall** de Linux. Maneja reglas que deciden qué tráfico 
de red se **acepta, rechaza o descarta**.

## 📋 Las "tablas" son conjuntos de reglas

Lo que viste antes son las **cadenas (chains)**:

| Chain | ¿Qué controla? |
|---|---|
| `INPUT` | Tráfico que **entra** al servidor |
| `OUTPUT` | Tráfico que **sale** del servidor |
| `FORWARD` | Tráfico que **pasa** por el servidor hacia otro |

## 🔍 Analogía simple

Imagina `iptables` como un **guardia de seguridad** en la puerta 
del servidor:
```
Internet → [iptables/INPUT] → Servidor
                ↓
    ¿Está el puerto en la lista?
    ✅ SÍ → ACCEPT (entra)
    ❌ NO → REJECT (bloqueado)
```


**Paso 1. Identificar el servidor que no renderiza la página**  

Desde el Jump Host, hacer la péticion curl:

El servidor que generó el error fue _stapp01_

```
thor@jump-host ~$ curl http://stapp01:8082
curl: (7) Failed to connect to stapp01 port 8082: No route to host
thor@jump-host ~$ 
```
**Paso 2. Conectar al server y verificar el estado del servicio**  

```
[root@stapp01 ~]# systemctl status httpd
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Thu 2026-03-26 12:54:42 UTC; 8min ago
       Docs: man:httpd.service(8)
    Process: 16370 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 16370 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 34ms

Mar 26 12:54:42 stapp01 systemd[1]: Starting The Apache HTTP Server...
Mar 26 12:54:42 stapp01 httpd[16370]: AH00558: httpd: Could not reliably determine the server's fully quali>
Mar 26 12:54:42 stapp01 httpd[16370]: (98)Address already in use: AH00072: make_sock: could not bind to add>
Mar 26 12:54:42 stapp01 httpd[16370]: (98)Address already in use: AH00072: make_sock: could not bind to add>
Mar 26 12:54:42 stapp01 httpd[16370]: no listening sockets available, shutting down
Mar 26 12:54:42 stapp01 httpd[16370]: AH00015: Unable to open logs
Mar 26 12:54:42 stapp01 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Mar 26 12:54:42 stapp01 systemd[1]: httpd.service: Failed with result 'exit-code'.
Mar 26 12:54:42 stapp01 systemd[1]: Failed to start The Apache HTTP Server.
lines 1-18/18 (END)
```

Se identifica la falla en la línea
```
 Active: failed (Result: exit-code) since Thu 2026-03-26 12:54:42 
```

Se intenta iniciar el servicio, sin embargo genera error

```
[root@stapp01 ~]# systemctl start httpd
Job for httpd.service failed because the control process exited with error code.
See "systemctl status httpd.service" and "journalctl -xeu httpd.service" for details.
[root@stapp01 ~]# 
```

Se utiliza el siguiente comando para ver en detalle el estado del servicio.
```
systemctl status httpd.service
```

salida

```
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Thu 2026-03-26 13:09:03 UTC; 9min ago
       Docs: man:httpd.service(8)
    Process: 43797 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 43797 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 33ms

Mar 26 13:09:03 stapp01 systemd[1]: Starting The Apache HTTP Server...
Mar 26 13:09:03 stapp01 httpd[43797]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.244.202. Set the 'ServerName' directive globall>
Mar 26 13:09:03 stapp01 httpd[43797]: (98)Address already in use: AH00072: make_sock: could not bind to address [::]:8082
Mar 26 13:09:03 stapp01 httpd[43797]: (98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:8082
Mar 26 13:09:03 stapp01 httpd[43797]: no listening sockets available, shutting down
Mar 26 13:09:03 stapp01 httpd[43797]: AH00015: Unable to open logs
Mar 26 13:09:03 stapp01 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Mar 26 13:09:03 stapp01 systemd[1]: httpd.service: Failed with result 'exit-code'.
Mar 26 13:09:03 stapp01 systemd[1]: Failed to start The Apache HTTP Server.
```

La clave se encuentra en la línea, el puerto 8082 se encuentra utilizado

```
(98)Address already in use: AH00072: make_sock: could not bind to address [::]:8082
```

Se procede a identificar que servicio esta utilizando en puerto 8082

```
[root@stapp01 ~]# ss -tlnp | grep 8082
LISTEN 0      10         127.0.0.1:8082      0.0.0.0:*    users:(("sendmail",pid=15800,fd=4))
[root@stapp01 ~]# 
```
De acuerdo al resultado el servicio sendmail (con pid 15800)está utilizando el puerto.

Procedemos a finalizar de manera forzada ese proceso para liberar el puerto.

`kill -9 PIDSIGKILL` Fuerza la terminación inmediata. El proceso no puede ignorarla

```
[root@stapp01 ~]# kill -9 15800
[root@stapp01 ~]# ss -tlnp | grep 8082
[root@stapp01 ~]# 
```
Procedemos a iniciar el servicio de Apache, en esta ocasión no genera error.

```
[root@stapp01 ~]# systemctl start httpd
[root@stapp01 ~]# 
```

Intentamos el curl de nuevo

```
thor@jump-host ~$ curl http://stapp01:8082
curl: (7) Failed to connect to stapp01 port 8082: No route to host
thor@jump-host ~$
```
Sin embargo generó error. 

Se requiere volver al servidor y verificar el estado de `iptables` = _Tablas de reglas para filtrar tráfico IP_

```
# iptables -L -n | grep 8082
```
Al no devolver nada, nos indica que no existe una regla para el puerto 8082.

El firewall SÍ está activo y bloqueando

Mira esta línea al final de INPUT:
```
REJECT  all  --  0.0.0.0/0  0.0.0.0/0  reject-with icmp-host-prohibited
```
Esto significa "rechaza todo el tráfico que no esté explícitamente permitido". Y solo el puerto 22 (SSH) está permitido. El puerto 8082 no está en la lista.

**Agregar una regla que permita el puerto 8082 antes de la regla REJECT**

```
iptables -I INPUT -p tcp --dport 8082 -j ACCEPT
```

### 🧠 ¿Por qué `-I` y no `-A`?

| Opción | Significado |
|---|---|
| `-A` | **Agrega** al final — quedaría DESPUÉS del REJECT, inútil |
| `-I` | **Inserta** al inicio — queda ANTES del REJECT, funciona ✅ |

Se verifica de nuevo y ya se encuentra la regla para el puerto 8082

```
[root@stapp01 ~]# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8082
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
[root@stapp01 ~]# 
```

Se verifica de nuevo y ahora ya permite la conexion con curl

curl http://stapp01:8082


**Resumen del flujo:**
```


 ```
