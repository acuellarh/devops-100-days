# Day 15: Setup SSL for Nginx

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 1 in Stratos Datacenter. They have some pre-requites to get ready that server for application deployment. Prepare the server as per requirements shared below:

1. Install and configure nginx on App Server 1.

2. On App Server 1 there is a self signed SSL certificate and key present at location /tmp/nautilus.crt and /tmp/nautilus.key. Move them to some appropriate location and deploy the same in Nginx.

3. Create an index.html file with content Welcome! under Nginx document root.

4. For final testing try to access the App Server 1 link (via hostname) from jump host using curl command. For example: curl -Ik https://stapp01



**Conectarse al servidos e instalar nginx**  


```
ssh tony@stapp01 

```
Se procede a instalar nginx

```
sudo yum install -y nginx
```

**Posteriormente crear y mover los archivos SSL a una ruta establecida**

```
sudo mkdir -p /etc/nginx/ssl
sudo mv /tmp/nautilus.crt /etc/nginx/ssl/nautilus.crt
sudo mv /tmp/nautilus.key /etc/nginx/ssl/nautilus.key

```

**Ajustar los permisos de la clave privada que solo debe tener acceso desde el root**

```
sudo chmod 600 /etc/nginx/ssl/nautilus.key
```
**Crear el archivo index.html**

```
sudo bash -c 'echo "Welcome!" > /usr/share/nginx/html/index.html'
```

**Configurar Nginx para HTTPS (SSL)**
Editar el archivo de configuración de Nginx para habilitar SSL:

```
sudo vi /etc/nginx/nginx.conf
```

Encuentre el cloque server {}  (o adicione uno nuevo) and configure lo siguiente.

```
server {
    listen       443 ssl;
    listen       [::]:443 ssl;
    server_name  stapp03;

    ssl_certificate     /etc/nginx/ssl/nautilus.crt;
    ssl_certificate_key /etc/nginx/ssl/nautilus.key;

    root  /usr/share/nginx/html;
    index index.html;

    location / {
    }
}
```

Configuración explicada:

* listen 443 ssl → HTTPS runs on port 443
* ssl_certificate → ruta al certificado publico (.crt)
* ssl_certificate_key → ruta al certificado privado (.key)
* root → where Nginx busca los archivos a servir
* server_name → El nombre del servidor

**Probar la configuración de nginx**

```
sudo nginx -t

```
Debería aparecer lo siguiente

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful


**Iniciar & habilitar Nginx**

```
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

**Probar la conexión https desde el server jump**
Go back to the jump host and run:

```
thor@jump-host ~$ curl -Ik https://stapp01/
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Thu, 09 Apr 2026 00:25:21 GMT
Content-Type: text/html
Content-Length: 9
Last-Modified: Thu, 09 Apr 2026 00:07:11 GMT
Connection: keep-alive
ETag: "69d6edaf-9"
Accept-Ranges: bytes

```
Significado de las banderas del comano

* -I → Unicamente devuelva las cabeceras 
* -k → Omita la verificación SSL  

