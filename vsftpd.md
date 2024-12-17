Update and patch the server:

```shell
# Updating repositories and upgrading the OS:
sudo apt-get update && sudo apt-get dist-upgrade
```

# The following changes below will occur in `/etc/vsftpd.conf`:

VSFTPD (Very Secure FTP Daemon) is a widely-used, secure FTP (File Transfer Protocol) server, that can have a variety of misconfigurations that present security risks. In this guide we will highlight some and provide remediations. 

If you haven't installed the service already, do so by:
```bash
sudo apt-get install vsftpd -y
```

---

0) Verify that the service account generated for FTP cannot login:

The FTP service user account should not be permitted to login, you can see if this is the case in the `/etc/passwd` file where you will see a string much like:
```bash
# Example of an FTP user with an interactive shell of /bin/bash, can login
ftp:x:1001:1001:ftp,,,:/home/ftp:/bin/bash
[--] - [--] [--] [-----] [--------] [--------]
|    |   |    |     |         |        |
|    |   |    |     |         |        +-> 7. Login shell
|    |   |    |     |         +----------> 6. Home directory
|    |   |    |     +--------------------> 5. GECOS
|    |   |    +--------------------------> 4. GID
|    |   +-------------------------------> 3. UID
|    +-----------------------------------> 2. Password
+----------------------------------------> 1. Username

# This is an example of a service account that cannot login.
ftp:x:1001:1001:ftp,,,:/home/ftp:/usr/sbin/nologin
```

Make sure there is no interactive shell for that user account unless it is required to interact with the service, much how people will disable `root` where users with SUDO permissions replace the necessity. 

1) Disable Anonymous Access:

Allowing anonymous FTP access (an account literally named `anonymous`) is a significant security risk, as it can lead to unauthorized access to your files:
```bash
# Disable FTP Anonymous Access
anonymous_enable=NO
```

2) Enable Local User Accounts:

Local users are accounts that exist on the system, and by enabling them, you can allow authenticated users to access the FTP server (probably makes sense in a production environment, defer to readme):
```bash
# Allow local users to login;
local_enable=yes
```

3) Restrict Users to Their Home Directories:

To prevent users from accessing directories outside of their home directory, you can set `chroot_local_user` to 'YES', which ill confine them to their own directories:
```bash
# Lock users into their home directories
chroot_local_users=YES
```

4) Disable FTP Write Access for Users:

To limit the risk of users modifying or uploading files, you can disable write permissions. If write access is required for certain users or directories, this can be adjusted accordingly:
```bash
# Disable FTP write access for users
write_enable=NO
```

If you need to allow writes for specific users or directories, you can configure them using `local_umask` or by enabling write access only for specific user groups.

Source: [ftp - vsftpd: allow access only for certain users - Server Fault](https://serverfault.com/questions/243816/vsftpd-allow-access-only-for-certain-users) 
```bash
> In vsftpd.conf add:  
> `userlist_enable=YES`  
> `userlist_file=/etc/vsftpd.userlist`  
> `userlist_deny=NO`

Edit the file to contain a username per row.
```

5) Use Passive Mode (Configure Ports)

In passive mode, the FTP client initiates both the data connection and the control connection. This is useful when the server is behind a firewall or NAT. However, it can require a specific range of ports to be open:
```bash
# Define Passive Mode Port Range
pasv_min_port=30000
pasv_max_port=31000

# Set the passive mode IP (optional, only needed if behind NAT)
pasv_address=your.server.ip.address
```

Ensure the port range is allowed through the firewall

For the competition where data transfers do not need to occur (defer to readme), you can disable passive mode outright:
```bash
# Disable Passive Mode
pasv_enable=NO
```

6) Use Secure Connections (TLS/SSL)

FTP transmits data, including passwords, in plain text. TLS/SSL ensures that the connection bewteen the client and server is encrypted, preventing eavesdropping and man-in-the-middle attacks.

You will need to generate SSL certificates first:
```bash
sudo openssl genpkey -algorithm RSA -out /etc/ssl/private/vsftpd.key

sudo openssl req -new -key /etc/ssl/private/vsftpd.key -out /etc/ssl/private/vsftpd.csr

sudo openssl x509 -req -in /etc/ssl/private/vsftpd.csr -signkey /etc/ssl/private/vsftpd.key -out /etc/ssl/private/vsftpd.crt
```

Once you have the certificate and key files you can enable TLS in the configuration file:
```bash
# Enable SSL and Ciphers
ssl_enable=YES
ssl_ciphers=HIGH

# Enable TLS (TLS is more secure than SSL)
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO

# Point to the certificate and key files
rsa_cert_file=/etc/ssl/private/vsftpd.crt
rsa_private_key_file=/etc/ssl/private/vsftpd.key

# Optionally, enable TLS for data connections
force_local_data_ssl=YES
force_local_logins_ssl=YES
```

These settings force encrypted connections for both login and data transfers.

8) Limit FTP Access to Specific IP addresses:

To minimize exposure to attacks, limit FTP access to trusted IP addresses or subnet. This can be done using `tcp_wrappers` or by configuring the firewall:
```bash
# Allow only specific IP addreses or networks to connect
hosts_allow=192.168.1.0/24
hosts_deny=ALL
```

This configuration ensures that only users from the 192.168.1.0/24 subnet can access the FTP server, while all other connections are denied.

9) Limit Concurrent Connections:

By limiting the number of concurrent connections from a single client, you can prevent abuse or denial of service:
```bash
# Limit the number of simultaneous connections from a single IP
max_per_ip=5
```

10) Disable Unnecessary FTP Commands:

If you're only using FTP for file retrieval (download), you can disable file upload and other unnecessary commands:
```bash
# Diasable the command to delete files (import for security)
cmds_denied=DELE,RMD,STOU

# Disable anonymous upload and directory writing
anon_upload_enable=NO
anon_mkdir_write_enable=NO
```
Source: [linux - Disable upload in vsftpd - Server Fault](https://serverfault.com/questions/753072/disable-upload-in-vsftpd)

11) Enable Logging

Logging is essential for tracking access to FTP servers and detecting suspcious activities:
```bash
# Enable logging and set it to capture useful information
xferlog_enable=YES
log_ftp_protocol=YES
xferlog_file=/var/log/vsftpd.log
```

This configuration will log FTP transfer activities to `/var/log/vsftpd.log`.

12) Set Directory Permissions for the FTP Server:

Ensure that the file permissions on the FTP server are configured correctly to prevent unauthorized access to sensitive files:
```bash
# Use chmod and chown to set proper permissions
sudo chmod 750 /var/ftp
sudo chown root:ftp /var/ftp
```

This ensures that the FTP directory is accessible only by the root user and users in the ftp group.

13) Set File Permissions for the FTP Server:

```bash
# Set the umask for uploaded files to 022 (default rw-r--r--)
local_umask=022

# Set permissions for new files
file_open_mode=0666

# Restrict the FTP root for local users
local_root=/home/ftp
```

---
Use AppArmor on SELinux:
--
AppArmor enforces security policies for applications, including restricting their access to files, directories, and resources.

1) Install the AppArmor Utilities:

```bash
# Installing
sudo apt-get install apparmor-utils
```

2) Enable AppArmor for VSFTPD:

If you're using a default VSFTPD profile, AppArmor should already have a profile for vsftpd. You need to enforce this profile to restrict the application based on its rule.

To enforce the vsftpd profile, run the following:
```bash
sudo aa-enforce /etc/apparmor.d/usr.sbin.vsftpd
```

This will load the profile into enforce mode, meaning any violations of the defined rules will be blocked and logged. The profile defines which directories VSFTPD is allowed to access and what actions it can perform.

3) Verify AppArmor Profile Status

To confirm that the AppArmor profile is loaded and enforced, use the following:
```bash
sudo aa-status
```

This will list the current profiles, and you should see vsftpd under the "enforced" section. It should look like this:
```bsah
/usr/sbin/vsftpd
	status: enforce
```
