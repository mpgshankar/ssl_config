# ssl_config
used for ssl configuration

# NGINX SSL_CONFIG using Certbot
## Install & Configure Nginx
### Step 1: Install Nginx on Ubuntu
```sh
sudo apt-get update
sudo apt-get install nginx
```
### Step 2: Adjust the Firewall

Before we can test Nginx, we need to reconfigure our firewall software to allow access to the service. Nginx registers itself as a service with ufw, our firewall, upon installation. This makes it rather easy to allow Nginx access.
We can list the applications configurations that ufw knows how to work with by typing:
```sh
sudo ufw app list
```
You should get a listing of the application profiles:
```sh
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
As you can see, there are three profiles available for Nginx:

* Nginx Full: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
* Nginx HTTP: This profile opens only port 80 (normal, unencrypted web traffic)
* Nginx HTTPS: This profile opens only port 443 (TLS/SSL encrypted traffic)
It is recommended that you enable the most restrictive profile that will still allow the traffic you've configured. Since we haven't configured SSL for our server yet, in this guide, we will only need to allow traffic on port 80.

You can enable this by typing:
```sh
sudo ufw allow 'Nginx HTTP'
```
You can verify the change by typing:
```sh
sudo ufw status
```
You should see HTTP traffic allowed in the displayed output:
```sh
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```
### Step 3: Check your Web Server
At the end of the installation process, Ubuntu 16.04 starts Nginx. The web server should already be up and running.

We can check with the ```sh__systemd__``` init system to make sure the service is running by typing:
```sh
systemctl status nginx
```
```sh
Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2016-04-18 16:14:00 EDT; 4min 2s ago
 Main PID: 12857 (nginx)
   CGroup: /system.slice/nginx.service
           ├─12857 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
           └─12858 nginx: worker process
```     

When you have your server's IP address or domain, enter it into your browser's address bar:
```sh
http://server_domain_or_IP
```
You should see the default Nginx landing page, which should look something like this:

<p align="center">
    <img width="70%" height="80%" src="https://camo.qiitausercontent.com/c351da7e9719052ff64a6644b5b3fef8fd087061/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3133373033312f33373139323564632d306663332d336638352d636536302d3234643362653032366635352e706e67" >
</p>

### Step 4: Manage the Nginx Process
Now that you have your web server up and running, we can go over some basic management commands.

To stop your web server, you can type:
```sh
sudo systemctl stop nginx
```
To start the web server when it is stopped, type:
```sh
sudo systemctl stop start
```
To stop and then start the service again, type:
```sh
sudo systemctl restart nginx
```
If you are simply making configuration changes, Nginx can often reload without dropping connections. To do this, this command can be used:
```sh
sudo systemctl reload nginx
```

### Step 5: Configure Nginx with your <domain-name> and expose it on HTTPS
Go to path: 
```sh 
cd /etc/nginx/conf.d/ 
```
Open File: 
```sh 
sudo nano default.conf 
```
Change the content in default.conf as per the requirement:
```ssh
server {
    if ($host = <domain-name>) {
        return 301 https://$host$request_uri;
    } 

    listen 80;
        server_name <domain-name> <domain-name>;
    location / {
        proxy_pass http://$host:$port;   # host and port to be exposed externally on port 8080 
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
  return https://$host$request_uri;


}

server {

    listen 443;
    server_name <domain-name> <domain-name>; # Expose as https
    ssl_certificate <path-of-ssl-certificate-file>;
    ssl_certificate_key <path-of-ssl-certificate-file-key>;

    ssl on;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers AES256+EECDH:AES256+EDH:!aNULL;

    charset utf-8;
    access_log off;
    error_log  /var/log/nginx/localhost-error.log error;

    location / {
        proxy_pass http://$host:$port;   # host and port to be exposed externally on port 443 with SSL certificates
        proxy_http_version 1.1;  
        proxy_set_header Upgrade $http_upgrade;  
        proxy_set_header Connection 'upgrade';  
        proxy_set_header Host $host;  
        proxy_cache_bypass $http_upgrade; 
        add_header Front-End-Https on;
    }

}

```

Next test the above configuration of nginx
```sh
sudo nginx -t
```
```sh
Output:
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Now reload the nginx service
```sh
sudo service nginx reload
```



## Install Certbot
Certbot is part of EFF’s effort to encrypt the entire Internet. Secure communication over the Web relies on HTTPS, which requires the use of a digital certificate that lets browsers verify the identity of web servers (e.g., is that really google.com?). Web servers obtain their certificates from trusted third parties called certificate authorities (CAs). Certbot is an easy-to-use client that fetches a certificate from Let’s Encrypt—an open certificate authority launched by the EFF, Mozilla, and others—and deploys it to a web server.

### Step 1: Install Certbot on Ubuntu
```sh
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```
### Step 2: Generate SSL certificates for <domain-name> mentioned in Nginx configuration.
```sh
sudo certbot --nginx
```
```sh
Output:
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org

Which names would you like to activate HTTPS for?
-------------------------------------------------------------------------------
1: <domain-name>
-------------------------------------------------------------------------------
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
```
Once the <domain-name> is selected select option Redirect.
Once the certificates are generated there will be a Congratulation message.

### If certificates which are already generated and are misplaced than follow the below step.

```sh
sudo certbot certonly -d <domain-name>
```
