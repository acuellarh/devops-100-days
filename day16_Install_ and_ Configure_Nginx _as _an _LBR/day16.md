# Day 16: Install and Configure Nginx as an LBR

## Details 
https://engineer.kodekloud.com/project-details
## Actividad

Day by day traffic is increasing on one of the websites managed by the Nautilus production support team. Therefore, the team has observed a degradation in website performance. Following discussions about this issue, the team has decided to deploy this application on a high availability stack i.e on Nautilus infra in Stratos DC. They started the migration last month and it is almost done, as only the LBR server configuration is pending. Configure LBR server as per the information given below:

a. Install nginx on the LBR (load balancer) server if it is not already installed.

b. Configure load-balancing with the http context making use of all App Servers. Ensure that you update only the main Nginx configuration file located at /etc/nginx/nginx.conf.

c. Make sure you do not update the apache port that is already defined in the apache configuration on all app servers, also make sure apache service is up and running on all the app servers.

d. Once done, you can access the website by running curl http://stlb01:80 in the terminal.

## Context

What is a Load Balancer?
A Load Balancer (LBR) distributes incoming traffic across multiple servers so no single server gets overwhelmed. Think of it like a traffic cop directing cars to different lanes.

```
Internet → [Nginx LBR :80] → App Server 1
                           → App Server 2  
                           → App Server 3

```

Nginx acts as the "front door" — it receives all requests and forwards them to the app servers (which run Apache).

## Solution steps

**Find the Apache ports on the App Servers**  

Before configuring Nginx, you need to know which port Apache is listening on in each app server (stapp01, stapp02, stapp03). SSH into each one:

```
# On each app server (run separately)
ssh tony@stapp01       # App Server 1
ssh steve@stapp02      # App Server 2
ssh banner@stapp03     # App Server 3
```
Once inside each server, check the Apache config:

```
# Check Apache port (could be 8080 or another port — NOT 80)
grep -i "^Listen" /etc/httpd/conf/httpd.conf

# Also confirm Apache is running
sudo systemctl status httpd
```
Fot this case the port assigned is _3200_

**SSH into the LBR server**

```
ssh loki@stlb01
```

**Install Nginx (if not installed)**

```
# Check if nginx is already installed
nginx -v

# If not installed:
sudo yum install -y nginx      # For CentOS/RHEL
# or
sudo apt install -y nginx      # For Ubuntu/Debian
```
** Edit the Nginx configuration**

This is the core part. You only edit /etc/nginx/nginx.conf.

```
sudo vi /etc/nginx/nginx.conf
```

You need to add an upstream block and a server block inside the http {} section. Here's what the relevant section should look like:

```
http {
    # --- ADD THIS: defines the pool of app servers ---
    upstream myapp {
        server stapp01:8080;   # App Server 1 (use the port you found in Step 1)
        server stapp02:8080;   # App Server 2
        server stapp03:8080;   # App Server 3
    }

    # --- ADD THIS: the virtual server that listens on port 80 ---
    server {
        listen 80;

        location / {
            proxy_pass http://myapp;   # forward traffic to the upstream pool
        }
    }

    # ... leave any existing lines (log_format, etc.) intact ...
}
```

What each part does:

`upstream myapp`: Defines the group of backend servers. Nginx uses Round Robin by default (sends each new request to the next server in turn).

`server { listen 80; }`: Makes Nginx listen on port 80 for incoming traffic

`proxy_pass http://myapp`: Forwards the request to one of the servers in the upstream pool

For this step i used tee command in the following way

```
sudo tee /etc/nginx/nginx.conf > /dev/null << 'EOF'
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include             /etc/nginx/conf.d/*.conf;

    upstream myapp {
        server stapp01:8080;
        server stapp02:8080;
        server stapp03:8080;
    }

    server {
        listen       80;
        location / {
            proxy_pass http://myapp;
        }
    }
}
EOF
```

**Test and start Nginx**

```
# Test config for syntax errors — always do this before restarting!
sudo nginx -t

# If output says "syntax is ok" and "test is successful", then:
sudo systemctl start nginx
sudo systemctl enable nginx    # auto-start on reboot
```

**Verify it works**

```
# Run this from any terminal connected to the environment
curl http://stlb01:80
```
You should see the HTML response from one of the app servers. Run it multiple times — you'll notice responses may come from different servers (Round Robin in action! 🔄).

The final process of the request is the following.

```
1. A user sends:   GET http://stlb01:80/index.html

2. Nginx receives the request on port 80

3. The location / block matches (because / matches everything)

4. proxy_pass http://myapp tells Nginx:
      → look up the upstream pool called "myapp"
      → pick the next available server (Round Robin)
      → forward the full request to it

5. For example, Nginx forwards to:  http://stapp02:8080/index.html

6. Apache on stapp02 processes it and sends back a response

7. Nginx receives the response and sends it back to the user

```























