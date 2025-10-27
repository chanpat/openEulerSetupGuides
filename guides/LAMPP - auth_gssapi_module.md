# Apache mod_auth_gssapi (Kerberos SSO) on LAMPP
**A step-by-step installation and configuration guide for Apache mod_auth_gssapi on openEuler Linux.**  
**This guide assumes that LAMPP stack has been installed in the system.**

## Install Required Packages
```
yum install mod_auth_gssapi krb5-client
```

#### Verify Installation
```
rpm -q mod_auth_gssapi krb5-client
klist -V
```

## Kerberos Configuration

Edit `/etc/krb5.conf` and configure your realm:
```
[libdefaults]
    default_realm = ABC.ORG.MO
    dns_lookup_realm = false
    dns_lookup_kdc = true
    rdns = false
    forwardable = true
    default_ccache_name = KEYRING:persistent:%{uid}

[realms]
    ABC.ORG.MO = {
        kdc = FMDC01.abc.org.mo
        kdc = FMDC02.abc.org.mo
        admin_server = FMDC01.abc.org.mo
        default_domain = abc.org.mo
    }

[domain_realm]
    .abc.org.mo = ABC.ORG.MO
    abc.org.mo = ABC.ORG.MO
```
> **Important:** Remove the line `includedir /etc/krb5.conf.d/` to avoid repeated credential prompts.

## Active Directory Service Account Setup

1. **Create AD service account:**
   - AD New user: `http-openeuler`
2. **Register ServicePrincipalName (SPN):**
   - cmd: `setspn -S HTTP/openeuler.abc.org.mo http-openeuler`
3. **Verify SPN registration:**
   - cmd: `setspn -L http-openeuler`
   - cmd: `setspn -Q HTTP/openeuler.abc.org.mo`
4. **Remove SPN (if needed):**
   - cmd: `setspn -D HTTP/openeuler.abc.org.mo http-openeuler`

## Generate Kerberos Keytab

Use `ktutil` to generate `/etc/http.keytab`:
```
ktutil
ktutil: addent -password -p HTTP/openeuler.abc.org.mo@ABC.ORG.MO -k 1 -e aes256-cts-hmac-sha1-96
ktutil: [Enter AD service account password]
ktutil: addent -password -p HTTP/openeuler.abc.org.mo@ABC.ORG.MO -k 1 -e rc4-hmac
ktutil: [Enter AD service account password]
ktutil: addent -password -p http-openeuler@ABC.ORG.MO -k 1 -e aes256-cts-hmac-sha1-96
ktutil: [Enter AD service account password]
ktutil: addent -password -p http-openeuler@ABC.ORG.MO -k 1 -e rc4-hmac
ktutil: [Enter AD service account password]
ktutil: list
ktutil: wkt /etc/http.keytab
ktutil: quit
```

#### Set Keytab Permissions
```
chown apache:apache /etc/http.keytab
```

#### Verify Keytab
```
klist -kte /etc/http.keytab
```

## Test Kerberos Authentication

- **User principal (Apache → AD):**
  ```
  kinit -kt /etc/http.keytab http-openeuler@ABC.ORG.MO
  klist
  ```
- **Service principal (Client → Apache):**
  ```
  kvno HTTP/openeuler.abc.org.mo@ABC.ORG.MO
  ```

## Enable mod_auth_gssapi in Apache (XAMPP)
Find install location of mod_auth_gssapi.so:
```
rpm -ql mod_auth_gssapi | grep .so
```
Expected output:
```
/usr/lib64/httpd/modules/mod_auth_gssapi.so
```

Edit `/etc/httpd/conf.modules.d/00-base.conf` and add:
```
LoadModule auth_gssapi_module modules/mod_auth_gssapi.so
```

## Configure GSSAPI Authentication for a Directory
Edit `/etc/httpd/conf/httpd.conf`  
>Example for `/var/www/html/domain`:
```
...
<Directory "/var/www/html/domain">
    AllowOverride all
    Options None
    Order allow,deny
    Allow from all

    AuthType GSSAPI
    AuthName "GSSAPI Single Sign-On Login"
    GssapiCredStore keytab:/etc/http.keytab

    Require valid-user
</Directory>
...
```

## Restart Apache (XAMPP)
```
systemctl restart httpd
```

## Verify Module is Enabled
```
apachectl -M | grep gssapi
```
Expected output:
```
auth_gssapi_module (shared)
```
## Add a Host (A) Resource Record in DNS server
Name  | Type  | IP 
:---  | :---  | :--- 
openeuler | A | 192.168.50.244<br>(openEuler server IP)
## Test SSO on web browser(MS Edge/Chrome)
create `/var/www/html/domain/index.php` 
```
<?php
echo "Remote User: " . ($_SERVER['REMOTE_USER'] ?? 'NOT SET');
?>
```
Visit `http://openeuler/domain`  
Expected output:
```
Remote User: logged_in_username@ABC.ORG.MO
```
