Update and patch the server:

```shell
# Updating repositories and upgrading the OS:
sudo apt-get update && sudo apt-get dist-upgrade
```

# The following changes below will occur in `/etc/chrony/chrony.conf:

Chrony is typically preferred over traditional NTP for better accuracy and security, but NTP can work fine too:
```bash
# Install Chrony for NTP services
sudo apt-get install chrony -y

# To install traditional NTP (if not)
sudo apt-get install ntp -y
```

The configuration principles stay consistent with both, regardless of what you use.

1) Setting Trusted Time Sources:

Ensure you're using trusted and public NTP servers. Avoid using arbitrary of potentially insecure sources. You can use the `pool.ntp.org` server, or you could specify your own trusted NTP server:
```bash
# Use public NTP pool servers (recommended)
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```

You can also specify local or trusted servers if you're in a controlled environment, like the competition, which is common:
```bash
# Maybe this points to a cyber specific endpoint
server time.example.com iburst
```

2) Disable unnecessary access to NTP (chrony) service:

Limit access to NTP to only trusted hosts or local interfaces (prevents unauthorized NTP requests from external sources):
```bash
# Allow only local system (loopback) to access NTP service
allow 127.0.0.1

# Optionally, specify specific trusted subnets:
#allow 192.168.0.0/24 # for example
```

3) After configuring Chrony, enable and start the service:
```bash
# Enable and start the service:
sudo systemctl enable chronyd
sudo systemctl start chronyd

# Verify it is working with the synchonization command:
chronyc tracking
```

----
# The following changes below will occur in `/etc/ntp.conf:

1) Set Trusted Time Servers:

Similar to chrony, configure trusted NTP servers:
```bash
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst
server 3.pool.ntp.org iburst
```

2) Disable unnecessary NTP options:

Disable "restrict" statements that may expose your server to malicious time queries. For additional security, ensure that `ntpd` only allows times queries from trusted hosts or networks. YOu cna restrict access setting proper restrict parameters;
```bash
# A setup to restrict options
restrict default kod notrap nomodify nopeer noquery
restrict 127.0.0.1
restrict ::1
```

3) Ensure the service is running:

Start and enable NTP services:
```bash
sudo systemctl enable ntp
sudo systemctl start ntp

# Verify NTP synchronization
ntpq -p
```

4) Firewall configuration for NTP:

Ensure that NTP (port 123) is allowed only for the trusted IPs or local machines that need to sync time:
```bash
sudo ufw allow from 192.168.1.0/24 to any port 123 proto udp
sudo ufw enable
```

