# XAMPP Stack on openEuler
**A step-by-step installation and configuration guide for XAMPP on openEuler Linux**

## Disable Firewall
```
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```
> Note: Disabling firewall is for development environments only. For production, configure appropriate firewall rules instead.

## Install XAMPP

#### Transfer Installer
Use WinSCP or similar tool to copy the <a target="_blank" href="https://sourceforge.net/projects/xampp/files/XAMPP%20Linux/8.2.12/xampp-linux-x64-8.2.12-0-installer.run/download">XAMPP installer</a> to your server.
#### Make Installer Executable
```
chmod +x xampp-linux-x64-8.2.12-0-installer.run
```

#### Run the Installer
```
./xampp-linux-x64-8.2.12-0-installer.run
```

## Start and Verify XAMPP
```
/opt/lampp/lampp start
/opt/lampp/lampp status
```

#### Fix Common Startup Error
If you see a missing library error, install the required library:
```
yum install libnsl
/opt/lampp/lampp restart
```

## Configure phpMyAdmin Access

Edit `/opt/lampp/etc/extra/httpd-xampp.conf` to restrict access to phpMyAdmin:
```
...
<Directory "/opt/lampp/phpmyadmin">
    AllowOverride AuthConfig Limit
    Require ip 192.168.50.0/255.255.255.0
    Require ip 172.18.0.0/255.255.0.0
    Require local
    ErrorDocument 403 /error/XAMPP_FORBIDDEN.html.var
</Directory>
...
```
## Login phpMyAdmin
1. Login phpMyAdmin: `http://[openEuler IP Address]/phpmyadmin`
2. Set root@Localhost Password
3. Create pma account and set Password

## Configure phpMyAdmin Authentication

Edit `/opt/lampp/phpmyadmin/config.inc.php`:
```
...
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['controluser'] = 'pma';
$cfg['Servers'][$i]['controlpass'] = 'pma_password';
...
```

## Setup XAMPP as a Systemd Service (Autostart)

Create `/etc/systemd/system/xampp.service`:
```
[Unit]
Description=XAMPP Control Panel
After=network.target

[Service]
Type=forking
ExecStart=/opt/lampp/lampp start
ExecStop=/opt/lampp/lampp stop
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

Reload systemd and enable autostart:
```
sudo systemctl daemon-reload
sudo systemctl enable xampp.service
```
## Enable Case-Insensitive URLs in Apache

Edit `/opt/lampp/etc/httpd.conf`:
```
# Uncomment:
LoadModule speling_module modules/mod_speling.so
...
<Directory "/opt/lampp/htdocs">
    CheckSpelling on
    CheckCaseOnly on
    ...
</Directory>
```

## Configure MariaDB for Lowercase Table Names

Edit `/opt/lampp/etc/my.cnf` and add:
```
[mysqld]
lower_case_table_names = 1
```



## Log Files Locations

`/opt/lampp/logs/`

