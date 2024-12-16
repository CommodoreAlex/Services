Update and patch the server:
```bash
# Updating repositories and upgrading the OS:
sudo apt-get update && sudo apt-get dist-upgrade
```


# The following changes below will occur in `/etc/squid/squid.conf`

1) Limit access with ACLs (Access Control Lists):
```bash
# Restrict who can access the proxy based on IP address, domain, or network
# Allow only a specific IP range and deny any access that is not allowed
# Also review allowed ports and other services for what is / is not allowed
acl localnet src 192.168.1.0/24 # Define local network
http_access allow localnet      # Allow local network to use the proxy
http_access deny all            # Deny all other access
```

2) Configure authentication to limit proxy access to authorized users:
```bash
# Install Apache2-utils for 'htpasswd' utility:
sudo apt-get install apache2-utils -y

# Create a file to store authentication credentials
sudo touch /etc/squid/passwords

# Change the owner of the file to your authorized user
sudo chown proxy: /etc/squid/passwords

# Add a new user to Squid with the following command
sudo htpasswd -c /etc/squid/passwords username

# You will now be prompted to enter and confirm the password for the new user.
```

Configure Squid to use this file for authentication by editing `/etc/squid/squid.conf` to include:
```bash
# Below is pointing to the password file
auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid/passwords
acl authenticated proxy_auth REQUIRED # Require authentication for proxy usage
http_access allow authenticated       # Allow authenticated users
http_access deny all                  # Deny all non-authenticated access
```

3) Disable unnecessary features:
```bash
# Disable cache manager access
http_access deny manager
acl manager url_regex -i ^/manager

# Disable caching for sensitive content, if required:
cache_mem 8 MB
maximum_object_size_in_memory 4 KB
maximum_object_size 8 MB
```

4) Limit allowed HTTP methods:
```bash
# Block dangerous methods such as DELETE, PUT, etc:
acl safe_methods method GET POST HEAD
http_access allow safe_methods # Allow only GET, POST, and HEAD
http_access deny all           # Deny other undefined HTTP methods
```

5) Enable logging and monitoring activity:
```bash
# Enable access logging
access_log /var/log/squid/access.log squid

# You can check the logs with
sudo tail -f /var/log/squid/access.log
```

6) Implement firewall rules:
```bash
# Firewall settings are generally configured outside of Squid, with UFW/Iptables
sudo ufw allow from 192.168.1.0/24 to any port 3128 # Allow Squid on local network
# This does not require editing in the /etc/squid/squid.conf file
```

7) Use encryption (HTTPS proxying):
```bash
# You need SSL certs ready for it to actually function
https_port 3129 intercept ssl-bump cert=/etc/squid/cert.pem key=/etc/squid/cert.key
ssl_bump server-first all # Inspect SSL traffic
```

8) Limit resource usage:
```bash
maximum_object_size 10 MB  # Limit object size for cache
max_filedescriptors 2028   # Limit file descriptors
```

9) Securing configuration file permissions:
```bash
# Ensure that Squid is running under minimal privileges
sudo chmod 600 /etc/squid/squid.conf     # Secure Squid config file
sudo chmod 640 /etc/log/squid/access.log # Secure Squid log files

# Ensure the user running Squid has restricted permissions
sudo useradd -r -s /usr/sbin/nologin squid # No login shell (important), see /etc/passwd

# Change the 
```

10) Searching for rogue DNS entries in Squid's context:
```bash
# You should check the `/etc/hosts` file for rogue DNS entries too.
# Only allow DNS entries that are recommended by a readme file.
# Rogue DNS entries can also exist in `/etc/resolv.conf`
# Ensure the dns_nameservers directive points to trusted DNS servers
dns_nameservers 8.8.8.8 # Google's DNS
dns_nameservers 8.8.4.4 # Google's secondary DNS

# Flush the DNS cache to remove potentially poisoned entries:
sudo systemd-resolve --flush-caches
sudo service 

```

After editing the configuration file restart the services to apply the changes:
```bash
# Restarting the Squid service
sudo systemctl restart squid

# Test functionality by trying to access the proxy on the localhost
# You can see listening ports on the localhost with `ss -ntplu`
curl -x http://localhost:3128 http://google.com

# You can check Squid logs for any denied or malformed requests
sudo tail -f /var/log/squid/access.log
```

You're able to look or vulnerabilities in Squid or the underlying system with Nessus, OpenVAS, Lynis, and potentially Wuzzah:
```bash
# Tools like Lynis help with identifying vulnerabiltiies
sudo lynis audit system
```

Resources:
[Squid proxy configuration tutorial on Linux - LinuxConfig](https://linuxconfig.org/squid-proxy-configuration-tutorial-on-linux)
[How to Install and Configure Squid Proxy on Ubuntu 22.04 | Medium](https://medium.com/@redswitches/how-to-install-and-configure-squid-proxy-on-ubuntu-22-04-5755e726f846) 
