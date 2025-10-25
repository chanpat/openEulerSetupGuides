# Nginx as a Reverse Proxy on openEuler
**A step-by-step installation and configuration guide for Nginx as a Reverse Proxy on openEuler Linux**

**Linux:** OpenEuler 24.03 LTS SP2  
[**Download ISO**](https://repo.openeuler.org/openEuler-24.03-LTS-SP2/ISO/x86_64/openEuler-24.03-LTS-SP2-x86_64-dvd.iso)

## Disable Firewall
```
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```
> Note: Disabling firewall is for development environments only. For production, configure appropriate firewall rules instead.

## Install Nginx

#### Installation
```
yum install nginx -y
```

#### Verification
```
nginx -v
```
Expected output example:
```
nginx version: nginx/1.24.0
```

#### Start Nginx & Setup Autostart
```
systemctl start nginx
systemctl enable nginx
systemctl status nginx
```

## Configure Virtual Hosts

#### Create Custom Configuration
Nginx configurations are typically stored in `/etc/nginx/conf.d/`. Create a new configuration file:
```
nano /etc/nginx/conf.d/your-site.conf
```

#### Basic Virtual Host Example
```
server {
    listen 80;
    server_name www.example.com;
    
    root /var/www/html/example;
    index index.html index.htm index.php;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # PHP-FPM configuration (if using PHP)
    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

#### SSL/HTTPS Configuration Example
```
server {
    listen 443 ssl http2;
    server_name www.example.com;
    
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    root /var/www/html/example;
    index index.html index.htm index.php;
    
    location / {
        try_files $uri $uri/ =404;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name www.example.com;
    return 301 https://$server_name$request_uri;
}
```

> **Note:** Remove deprecated `ssl on;` directive if migrating from older configurations. Use `listen 443 ssl;` instead.

## Test and Apply Configuration
```
nginx -t
systemctl reload nginx
```

## Configure Log Rotation
Edit the logrotate configuration to adjust retention period:
```
nano /etc/logrotate.d/nginx
```

Change the rotation period (example: keep logs for 30 days):
```
/var/log/nginx/*.log {
    rotate 30
    ...
}
```

## View Nginx Logs
```
# Access logs
tail -f /var/log/nginx/access.log

# Error logs
tail -f /var/log/nginx/error.log
```

## Hide Nginx Version
   Add to `/etc/nginx/nginx.conf` in the `http` block:
   ```
   server_tokens off;
   ```

## Common Configuration Locations

- **Main Configuration:** `/etc/nginx/nginx.conf`
- **Virtual Host Configs:** `/etc/nginx/conf.d/*.conf`
- **Default Document Root:** `/usr/share/nginx/html`
- **Custom Document Roots:** `/var/www/html/`
- **Log Files:** `/var/log/nginx/`

## Testing with Custom Hosts File

For testing purposes, you can modify your local hosts file to point domain names to your Nginx server.

#### Windows
Edit the hosts file (run as Administrator):
```
notepad C:\Windows\System32\drivers\etc\hosts
```

Add entries (replace with your server IP):
```
192.168.50.244    www.example.com
192.168.50.244    app.example.com
192.168.50.244    api.example.com
```

Flush DNS cache:
```
ipconfig /flushdns
```



