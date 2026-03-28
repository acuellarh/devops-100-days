# Day 13: IPtables Installation And Configuration

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

We have one of our websites up and running on our Nautilus infrastructure in Stratos DC. Our security team has raised a concern that right now Apache’s port i.e 5003 is open for all since there is no firewall installed on these hosts. So we have decided to add some security layer for these hosts and after discussions and recommendations we have come up with the following requirements:

1. Install iptables and all its dependencies on each app host.
2. Block incoming port 5003 on all apps for everyone except for LBR host.
3. Make sure the rules remain, even after system reboot.

## Contexto

## 📚 Conceptos Clave — iptables

### Flags principales

| Término | Significado |
|---|---|
| `Chain INPUT` | Reglas para tráfico que **entra** al servidor |
| `-p tcp` | Protocolo TCP |
| `--dport 3002` | Puerto de **destino** 3002 |
| `-s [IP]` | **Source** (origen) — de dónde viene el tráfico |
| `-j ACCEPT/DROP` | **Jump** → qué acción tomar |

---

### ⚠️ ¿Por qué el orden importa?

iptables lee las reglas **de arriba hacia abajo** y aplica la primera que coincida.
Por eso hay que poner primero la excepción (permitir LBR) y luego el bloqueo general.
```
Regla 1 → ACCEPT al LBR       ← se evalúa primero
Regla 2 → DROP a todos         ← se evalúa si no coincidió con la 1
```

---

### 💾 ¿Qué es `iptables-save` / `iptables-services`?

Por defecto, las reglas de iptables **se pierden al reiniciar**.
Para hacerlas persistentes necesitas guardarlas:
```bash
# Guarda las reglas en disco
iptables-save > /etc/sysconfig/iptables
```

- `iptables-services` → servicio que **carga automáticamente** las reglas al arrancar
- `/etc/sysconfig/iptables` → archivo donde se **almacenan** las reglas guardadas



**Paso 1. Identificar la IP del equipo Load Balancer**  


```
thor@jump-host ~$ ssh loki@stlb01
loki@stlb01's password: 
[loki@stlb01 ~]$ hostname -I
10.244.248.2
[loki@stlb01 ~]$ 

```
**Paso 2. En cada app host instalar iptables y sus dependencias**  

verificar si esta instalado iptables

RPM = Red Hat Package Manager — es el sistema de gestión de paquetes de distribuciones como CentOS, RHEL, Fedora. Es el equivalente a apt en Ubuntu/Debian.

Analogía simple
Imagina que RPM es el inventario de un almacén. Con -q estás buscando en ese inventario:
¿Está iptables en el inventario? → rpm -q iptables

```
[tony@stapp01 ~]$ rpm -q iptables
package iptables is not installed
[tony@stapp01 ~]$ 
```

Se procede a instalar, iniciar, habilitar y verificar reglas
```
# 1. Instalar iptables y dependencias
sudo yum install -y iptables iptables-services

...resto código
Installed:
  iptables-legacy-1.8.10-11.1.el9.x86_64             iptables-legacy-libs-1.8.10-11.1.el9.x86_64   

# 2. Iniciar y habilitar el servicio
sudo systemctl start iptables
sudo systemctl enable iptables

# 3. Ver las reglas actuales (para entender el estado inicial)
sudo iptables -L --line-numbers -n
```

Verificar las reglas actuales

```
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
2    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
4    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
5    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         
1    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination    
```

Adicionar las nuevas reglas
```
# Regla 1: Permitir al LBR acceder al puerto 5003
iptables -I INPUT 1 -p tcp --dport 5003 -s 10.244.248.2 -j ACCEPT

# Regla 2: Bloquear a todos los demás en el puerto 5003
iptables -I INPUT 2 -p tcp --dport 5003 -j DROP
```

salida

```
[root@stapp01 ~]# iptables -L --line-numbers -n
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     tcp  --  10.244.248.2         0.0.0.0/0            tcp dpt:5003
2    DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:5003
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
4    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
5    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
6    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
7    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         
1    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
```

Siguiente paso — Guardar las reglas para que persistan

En CentOS 9 (el que usa este lab) el comando service fue reemplazado. Para guardar las reglas usa:

```
iptables-save > /etc/sysconfig/iptables
```
Confirmar persistencia
```
sudo cat /etc/sysconfig/iptables
```
salida
```
[root@stapp01 ~]# cat /etc/sysconfig/iptables
# Generated by iptables-save v1.8.10 on Sat Mar 28 22:22:35 2026
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [396:41008]
-A INPUT -s 10.244.248.2/32 -p tcp -m tcp --dport 5003 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 5003 -j DROP
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
# Completed on Sat Mar 28 22:22:35 2026
```



**Resumen del flujo:**

## 🔄 Flujo completo — iptables en Stratos DC
```
stlb01 (LBR)
ip addr show → 10.244.13.91
        |
        | (única IP autorizada)
        ↓
stapp01 / stapp02 / stapp03
        |
        ├── yum install iptables iptables-services
        |
        ├── systemctl start iptables
        ├── systemctl enable iptables
        |
        ├── iptables -I INPUT 1 -p tcp --dport 3002 -s 10.244.13.91 -j ACCEPT
        ├── iptables -I INPUT 2 -p tcp --dport 3002 -j DROP
        |
        ├── iptables -L INPUT --line-numbers -n  (verificar reglas)
        |
        ├── iptables-save > /etc/sysconfig/iptables (persistir)
        |
        └── cat /etc/sysconfig/iptables  (verificar que se guardó)
```
