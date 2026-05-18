---
heroImage: '/vps-web-server-setup.svg'
title: 'Setting Up a Web Server on Your VPS: Nginx and Apache'
description: 'A step-by-step guide to installing and configuring popular web servers on your VPS.'
pubDate: 'Apr 08 2026'
---

To host websites or web applications on your VPS, you need web server software to handle HTTP/HTTPS requests from visitors' browsers. The two undisputed leaders in this space are **Apache** and **Nginx** (pronounced "engine-ex").

This guide covers the basics of setting up both on an Ubuntu/Debian-based VPS.

---

## Nginx: The High-Performance Option

Nginx is designed for high concurrency and performance. It uses an asynchronous, event-driven architecture, making it exceptionally fast at serving static files and acting as a reverse proxy for applications (like Node.js, Python/Gunicorn, or PHP-FPM).

### 1. Installation

Update your package lists and install Nginx:

```bash
sudo apt update
sudo apt install nginx -y
```

Once installed, Nginx should start automatically. You can verify its status:

```bash
sudo systemctl status nginx
```

### 2. Firewall Configuration

If you are using UFW (Uncomplicated Firewall), you need to allow HTTP and HTTPS traffic:

```bash
sudo ufw allow 'Nginx Full'
```

### 3. Server Blocks (Virtual Hosts)

Nginx uses "Server Blocks" (similar to Apache's Virtual Hosts) to host multiple domains on a single server.

Create a directory for your website:
```bash
sudo mkdir -p /var/www/yourdomain.com/html
sudo chown -R $USER:$USER /var/www/yourdomain.com/html
```

Create a basic `index.html` file in that directory. Then, create a server block configuration file:

```bash
sudo nano /etc/nginx/sites-available/yourdomain.com
```

Add the following basic configuration:

```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/yourdomain.com/html;
    index index.html index.htm index.nginx-debian.html;

    server_name yourdomain.com www.yourdomain.com;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the configuration by creating a symlink to the `sites-enabled` directory:

```bash
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
```

Test the configuration for syntax errors and restart Nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

## Apache (HTTPD): The Flexible Veteran

Apache has been the standard web server for decades. It is highly modular and relies on `.htaccess` files for directory-level configuration, which is deeply ingrained in the PHP/WordPress ecosystem.

### 1. Installation

Install Apache:

```bash
sudo apt update
sudo apt install apache2 -y
```

### 2. Firewall Configuration

Allow traffic through UFW:

```bash
sudo ufw allow 'Apache Full'
```

### 3. Virtual Hosts

Create the directory structure:

```bash
sudo mkdir -p /var/www/yourdomain.com/html
sudo chown -R $USER:$USER /var/www/yourdomain.com/html
```

Create a Virtual Host configuration file:

```bash
sudo nano /etc/apache2/sites-available/yourdomain.com.conf
```

Add the configuration:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com
    DocumentRoot /var/www/yourdomain.com/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the site and restart Apache:

```bash
sudo a2ensite yourdomain.com.conf
sudo systemctl reload apache2
```

## Which Should You Choose?

*   **Choose Nginx if:** You are building modern web applications, serving a massive amount of static content, acting as a load balancer or reverse proxy, or running Node.js/Python apps.
*   **Choose Apache if:** You heavily rely on `.htaccess` files, are running legacy PHP applications, or require specific Apache modules.

Many modern setups actually use **both**: Nginx acts as the front-end reverse proxy serving static files quickly, while passing dynamic requests to Apache backend servers.
