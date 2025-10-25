# LAMP Stack & phpMyAdmin on openEuler
**a step by step Installation guide for LAMP Stack and phpMyAdmin on openEuler Linux**   

## Disable Firewall
```
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```
> Note: Disabling firewall is for development environments only. For production, configure appropriate firewall rules instead.
## Setup Apache
#### Installation
```
yum install httpd -y
```
#### Verification
```
httpd -v
```
#### Start Apache & setup autostart
```
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```
## Setup PHP
#### Install PHP and Required Extensions
```
yum install php php-cli php-fpm php-mysqlnd php-json php-gd php-ldap php-odbc php-pdo php-xml php-mbstring php-bcmath -y
```
#### Verification
```
php -v
```
#### Start php-fpm & setup autostart
```
systemctl start php-fpm
systemctl enable php-fpm
systemctl status php-fpm
```
# Apache PHP Configuration
#### Install text editor
```
yum install nano
```
#### Configure PHP Handler
```
nano /etc/httpd/conf.d/php.conf
```
Replace the current config with the following config:
```
<Files ".user.ini">
    <IfModule mod_authz_core.c>
        Require all denied
    </IfModule>
    <IfModule !mod_authz_core.c>
        Order allow,deny
        Deny from all
        Satisfy All
    </IfModule>
</Files>

DirectoryIndex index.php

<FilesMatch \.php$>
    SetHandler "proxy:unix:/run/php-fpm/www.sock|fcgi://localhost"
</FilesMatch>

<FilesMatch "\.(ini|log|sh|sql)$">
    Require all denied
</FilesMatch>

```
#### Optional: Case-Insensitive URLs
1. Enable spelling module:
    ```
    nano /etc/httpd/conf.modules.d/00-optional.conf
    ```
    Uncomment:
    ```
    LoadModule speling_module modules/mod_speling.so
    ```
2. Configure case-insensitive matching:
    ```
    nano /etc/httpd/conf/httpd.conf
    ```
    Add inside the document root directive:
    ```
    <Directory "/var/www/html">
        CheckSpelling on
        CheckCaseOnly on
        # ... existing configuration ...
    </Directory>
    ```
#### Restart Apache
```
systemctl restart httpd
```
#### Test Configuration
Visit http://[YOUR_SERVER_IP]/ in your browser. You should see the openEuler Linux Test Page.

## Setup MariaDB
#### Installation
```
yum install mariadb-server mariadb -y
```
#### Verification
```
mysql --version
```
#### Start MariaDB & setup autostart
```
systemctl start mariadb
systemctl enable mariadb
systemctl status mariadb
```
#### Security Configuration
```
mysql_secure_installation
```
Follow these steps during configuration:
```
- Enter current password for root (enter for none): Enter
- Switch to unix_socket authentication: n
- Change the root password? Y -> Then enter a strong new password twice
- Remove anonymous users? Y
- Disallow root login remotely? Y
- Remove test database and access to it? Y
- Reload privilege tables now? Y
```
#### Optional: Case-Insensitive Table Names
```
nano /etc/my.cnf
```
Add to the [mysqld] section:
```
[mysqld]
lower_case_table_names=1
```
#### Restart MariaDB
```
systemctl restart mariadb
```
#### Database Connection Test
Create a test PHP file:
```
nano /var/www/html/index.php
```
Add the following content (replace with your actual password):
```
<?php
$link = mysqli_connect('localhost', 'root', 'YOUR_PASSWORD_HERE');
if (!$link) {
    die('Could not connect: ' . mysqli_error());
}
echo 'Connected successfully';
mysqli_close($link);
?>
```
Visit http://[YOUR_SERVER_IP]/ to see "Database connected successfully!".

# Setup phpMyAdmin
#### Download and Extract
```
# Install tar if not exist
yum install tar

# Download phpMyAdmin
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.1/phpMyAdmin-5.2.1-all-languages.tar.gz

# Extract archive
tar -xzf phpMyAdmin-5.2.1-all-languages.tar.gz

# Move to installation directory
mv phpMyAdmin-5.2.1-all-languages /usr/share/phpmyadmin

```
#### Permission Configuration
```
# Set ownership and permissions
chown -R apache:apache /usr/share/phpmyadmin/
chmod -R 755 /usr/share/phpmyadmin/
chcon -R -t httpd_sys_content_t /usr/share/phpmyadmin/

# Create and configure temp directory
mkdir -p /usr/share/phpmyadmin/tmp
chown apache:apache /usr/share/phpmyadmin/tmp
chmod 777 /usr/share/phpmyadmin/tmp
```
#### Apache Configuration
```
nano /etc/httpd/conf.d/phpmyadmin.conf
```
Add the following config:
```
Alias /phpmyadmin /usr/share/phpmyadmin

<Directory /usr/share/phpmyadmin>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

<Directory /usr/share/phpmyadmin/tmp>
    Require all granted
</Directory>
```
#### Restart Apache
```
systemctl restart httpd
```
#### Access phpMyAdmin
Visit http://[YOUR_SERVER_IP]/phpmyadmin to access the phpMyAdmin login page.
