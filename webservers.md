Update and patch the server:

```shell
# Updating repositories and upgrading the OS:
sudo apt-get update && sudo apt-get dist-upgrade
```

Installing PHP and Apache (the two work together, double check readme):
```bash
sudo apt-get install apache2 php libapache2-mod-php
```

Ensure that Apache and PHP are installed correctly:
```bash
# Verify status
sudo systemctl status apache2

# Create a test PHP file to confirm functionality
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php

# Visit in browser or use curl to reach the page being hosted (delete test file after)
curl http://localhost/info.php
```

---
# The following changes below will occur in `/etc/php/7.x/apache2/php.ini:
The main PHP configuration file is `php.ini` but the location may vary, it is typically found in `/etc/php/7.x/apache2/php.ini` (adjust for the version of PHP where `7.x` is).

1) Disable PHP functions that can be used maliciously:

Disable dangerous PHP functions:
```bash
# These will prevent reverse shells being returned usually
disable_functions=exec,shell_exec,passthru,system,popen,curl_ exec,curl_multi_exec,parse_ini_file,show_source,proc_open,pcn tl_exec
```

2) Limit PHP memory and execution time:

Set limits to prevent runaway scripts and excessive resource consumption:
```bash
max_execution_time = 30
max_input_time = 30
memory_limit = 128M
```

3) Disable file uploads:

Disable file uploads to prevent file inclusion and remote code execution vulnerabilities:
```bash
file_uploads = Off
```

4) Disable display of errors:

Never expose PHP errors on live systems. They may reveal sensitive information about your application:
```bash
display_errors = Off
log_errors = On
```

5) Configure error logging:

Log errors to a file to keep track of isues without displaying them to users:
```bash
error_log = /var/log/php_erorrs.log
```

6) Restrict PHP scripts to specific directories:

Restrict PHP scripts to a specific directory or a set of directories to prevent directory traversal and access to critical system files:
```ash
open_basedir = /var/www/html:/tmp:/usr/share/php
```

7) Enable SAFE MODE (depreciated in PHP 5.4):

If you're using older versions of PHP, you can enable `safe_mode` to restrict their functionality of PHP scripts. However, it is depreciated and not available in PHP 5.4 and newer, so upgrading PHP is recommended (read the readme on what version is required):
```bash
safe_mode = On
```

8) Restrict PHP information leakage (version enumeration):

```bash
expose_php = Off
```

10) Disable Remote Code Execution:

```bash
allow_url_fopen = Off
allow_url_include = Off
```

---
Apache Security Hardening:
--

**The location of the configuration file: `/etc/apache2/apache2.conf`**

1) Disable Directory Listings:

Disable automatic directory listings to prevent unauthorized users from viewing the contenst of directories:
```bash
<Directory /var/www/html>
	Options -Indexes
</Directory>
```

This prevents Apache from showing directory contents when there is no index.html or index.php file.

2) Disable `.htaccess` Overrides (optional):

If you're not using `.htaccess` files, disable the ability for users to override server settings with them:
```bash
<Directory /var/www/html>
	AllowOverride None
</Directory>
```

This ensures that no configuration can be changed at the directory level through `.htacccess` files.

3) Hide Apache Version Information:

Prevent Apache from revealing its version in HTTP headers, which can help attackers exploit known vulnerabilities:
```bash
ServerTokens Prod
ServerSignature Off
```

* Server Tokens Prod: This limits the server information to just the word "Apache" and the version.
* ServerSignature Off: Disables the server version from appearing in error messages or on web pages.

4) Configure SSL/TLS (Optional but Recommended):

To create the SSL certificates required for the pointing to the locations:
[How To Create a Self-Signed SSL Certificate for Apache in Ubuntu 20.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-20-04) 

If you need to enable HTTPS, ensure that SSL is properly configured to secure communication; file to edit is at `/etc/apache2/sites-available/default-ssl.conf`:
```bash
SSLEngine on
SSLCertificateFile /etc/ssl/certs/my_cert.crt
SSLCertificateKeyFile /etc/ssl/private/my_key.key
```

Configure strong SSL settings (remove weak ciphers):
```bash
SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite HIGH:!aNull:!MD5
```

Enable HTTP Strict Transport Security (HSTS) for extra security:
```bash
Header always set Strict-Transport-Security "max-age=315360000; includeSubDomains; preload"
```

5) Prevent Access to Sensitive Files:

You should prevent direct access to files like `.htaccess`, `.env`, and others that may contain sensitive configuration data:
```bash
<Files ~ "^\.">
	Order Allow,Deny
	Deny from all
</Files>
```

This will prevent the web server from serving hidden files such as those mentioned above that may contain sensitive information.

Nginx Security Hardening:
--

**The location of the configuration file: `/etc/nginx/nginx.conf`**


1) Disable Nginx Version Information:

By default, Nginx will reveal its version in HTTP headers, which could give attackers useful information. Disable this by:
```bash
# Inside of the HTTP block:
server_tokens off;
```

This hides the Nginx version from HTTP headers, making it harder for attackers to know what version you're running.

2) Disable Directory Listing:

Nginx doesn't enable directory listing by default, but it's always a good practice to ensure its disabled. Add the following to prevent directory listings:
```bash
# Add this inside of the location block:
location / {
	autoindex off;
}
```

This disables directory browsing, ensuring that visitors can't list files in a directory without an index file.

3) Restrict Access to Sensitive Files:

Just as with Apache, you should restrict access to sensitive files like `env`, `.htpasswd`, or `.git`. This can prevent unauthorized access to your application's sensitive configuration:
```bash
# Add the following inside the server block:
location ~* \.(env|git|htpasswd) {
	deny all;
}
```

This denies access to any requests that try to access those mentions, protecting sensitive data.

4) Limit HTTP Methods:

Limit which HTTP methods are allowed. The most common and secure methods are GET, POST, and HEAD. Disable other methods such as PUT, DELETE, TRACE, and OPTIONS. Which are rarely needed and can be exploited:
```bash
# Add the following inside the server block:
limit_except GET POST HEAD {
	deny all;
}
```

This restricts access to only the necessary HTTP methods and denies other potentially dangerous ones.

5) Setup SSL/TLS (Optional but Recommended):

For instructions to create certificates: [Nginx: CSR & SSL Installation (OpenSSL)](https://www.digicert.com/kb/csr-ssl-installation/nginx-openssl.htm#:~:text=How%20to%20Generate%20a%20CSR%20for%20Nginx%20Using,generated%20.key%20file.%20...%206%20Install%20Certificate%20)
[Bing Videos](https://www.bing.com/videos/riverview/relatedvideo?q=nginx+how+to+create+self+signed+certificates&mid=B1D64113CD73ED4480BFB1D64113CD73ED4480BF&FORM=VIRE)


Just like Apache, you should secure your Nginx server with SSL/TLS to ensure encrypted communication. Here is a basic SSL configuration for Nginx:
```bash
server {
	listen 443 ssl;
	server_name yourdomain.com;
	
	ssl_certificate /etc/nginx/ssl/my_cert.crt;
	ssl_certificate_key /etc/nginx/ssl/my_key.key;

	ssl_protocols TLSv1.2 TLSv1.3;
	ssl_ciphers HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers on;

	# HSTS (HTTP Strict Transport Security)
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
}
```

Ensure you also redirect HTTP traffic to HTTPS:
```bash
server {
	listen 80;
	server_name yourdomain.com;

	return 301 https://$host$request_uri;
}
```

---
By applying these security measures, you'll reduce the attack surface of your web server and enhance system security. 

Make sure to restart all relevant services for Apache and Nginx after making changes.

