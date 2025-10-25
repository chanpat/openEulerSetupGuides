# Nginx Reverse Proxy on openEuler
**A step-by-step installation and configuration guide for Nginx Reverse Proxy on openEuler Linux**

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
yum install nginx
```

#### Verification
```
nginx -v
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

#### Example
```
server {
    listen         80;
    return 301 https://$host$request_uri;
}

server {
		listen      443 ssl;
		server_name www.your-site.com;

		ssl_certificate        /etc/nginx/server.chained.crt;
       	ssl_certificate_key    /etc/nginx/server.key;
		
		client_max_body_size 100m;	
		proxy_ssl_session_reuse on;
		ssl_protocols TLSv1.2 TLSv1.3; 

		location /admin/ {
			allow 192.168.0.0/16;
			allow 172.18.0.0/16;
 			deny all;

			proxy_pass http://172.18.52.19/admin/;		
       	}

		location / {
            proxy_pass http://172.18.52.19/;		
       	}

		location /test/ {
            proxy_pass http://172.18.52.24/test/;	
       	}
}
```

> **Note:** Remove deprecated `ssl on;` directive if migrating from older configurations. Use `listen 443 ssl;` instead.

## Hide Nginx Version
   Add to `/etc/nginx/nginx.conf` in the `http` block:
   ```
   server_tokens off;
   ```

## Optional: Configure Log Rotation
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

## Optional: Increase client_header_buffer for long request URL
   Add to `/etc/nginx/nginx.conf` in the `http` block:
   ```
   client_header_buffer_size 1m;
   ```

## Test and Apply Configuration
```
nginx -t
systemctl reload nginx
```

## Log Files Locations
/var/log/nginx/

## Testing with Custom Hosts File

For testing purposes, you can modify your local hosts file to point domain names to your Nginx server.

#### Windows
Edit the hosts file (run as Administrator):
```
notepad C:\Windows\System32\drivers\etc\hosts
```

Add entries (replace with your server IP):
```
[Nginx_IP_Address]    www.your-site.com
[Nginx_IP_Address]    app.your-site.com
```

Flush DNS cache:
```
ipconfig /flushdns
```



