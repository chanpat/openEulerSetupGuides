# PHP Sendmail Setup with msmtp on openEuler

**A step-by-step installation and configuration guide for sending email from PHP using msmtp on openEuler Linux.**  
**This guide assumes that XAMPP has been installed in the system.**  

## Install Required Packages
```
yum install tar gcc make automake autoconf nano gnutls-devel -y
```

## Download and Install msmtp

1. Download msmtp from the official site:
   [msmtp Download Page](https://marlam.de/msmtp/download/).   
   Use WinSCP or similar tool to copy the archive to the server.

2. Extract the archive:
   ```
   tar -xf msmtp-1.8.32.tar.xz
   ```

3. Compile and install:
   ```
   cd msmtp-1.8.32
   ./configure
   make
   make install
   ```

## Verify Installation
```
which msmtp
msmtp --version
```

## Configure msmtp

Create the configuration file:
```
nano /etc/msmtprc
```

Example configuration:
```
defaults
auth on
logfile /var/log/msmtp.log

account default
host 192.168.50.41
port 25
user user123
password Abc123456!
from user123@abc.org.mo

tls on
tls_starttls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
tls_certcheck off

protocol smtp
```
> **Note:** For self-signed certificates, you may need to disable certificate verification (`tls_certcheck off`). Use with caution.

## Set Permissions
```
chown daemon:daemon /etc/msmtprc
chmod 600 /etc/msmtprc
```
## Setup msmtp Log File
```
touch /var/log/msmtp.log
chmod 666 /var/log/msmtp.log
```

## Test msmtp from Terminal
```
echo -e "Subject: Test Email\nFrom: user123@abc.org.mo\n\nTest from terminal." | msmtp -C /etc/msmtprc -t userABC@gmail.com
```

## Configure PHP to Use msmtp

Edit your `php.ini` file:
```
sendmail_path = "/usr/local/bin/msmtp -C /etc/msmtprc -t"
```



