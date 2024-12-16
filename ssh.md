Update and patch the server:

```shell
# Updating repositories and upgrading the OS:
sudo apt-get update && sudo apt-get dist-upgrade
```

# The following changes below will occur in `/etc/sshd/sshd_config:


The best way to go about configuring this file is to `CTRL+F` (find) the entries relevant, if pre-existing, to make workflows more productive.

1) Disable ROOT login:

Allowing root login over SSH gives attackers direct access to the highest privileges on a system. It is ideal to have this disabled by default and enforce the login of authorized less-privileged users:
```bash
PermitRootLogin = no
```

2) Use SSH Key-Based Authentication (Disable Password Authentication):

Password-based authentication is vulnerable to brute-force attacks. SSH key pairs are much more secure and difficult to crack (research public and private keys). Disabling password authentication forces the use of SSH keys:
```bash
PasswordAuthentication no
ChallengeResposneAuthentication no
UsePAM no # Disable Pluggable Authentication Modules (PAM) in lieu of keys
```

If you are not using key-based authentication, you should enable **PasswordAuthentication**, which may be more standard for cyber competitions (try both for points):
```bash
# SSH Keys provide a higher standard of security than passwords.
# Try key-based for points first, then this if no points are given during scoring
PasswordAuthentication yes

# IF you are using Passwords then enable PAM as well
UsePAM yes
```

3) Change the default SSH port (protect against port knocking):

Changing the default port (22) to a non-standard port makes automated attacks (e.g. from bots or hackers) targeting port 22 less likely to succeed:
```bash
# Make the port greater than 1024 as remote network mapping tools without
# any command switches scan against ports 1-1024 automatically 
Port 2222
```

If you add the new port make sure your firewall is allowing the new port to include incoming or outgoing connections, based on your context in the competition.

4) Limit SSH access by IP address:

Restricting SSH access to specific IP addresses (e.g. your internal network or trusted range) can prevent unauthorized external access:
```bash
# Use the AllowUsers or AllowGroups directives to limit SSH access to specific
# users or groups from certain IPs
AllowUsers user1@192.168.1* user2@192.168.0.0/24

# You can restrict access to groups too
AllowGroups admin sshusers

# You can also use IPTables or UFW to accomplish tis
sudo ufw allow from 192.168.1.0/24 to any port 2222 # Only allow from a specific subnet
sudo ufw deny from any to any port 2222 # Deny SSH from everywhere else
```

5) Enforce strong encryption algorithms:

Older SSH encryption algorithms (e.g. 3DES) are weak and vulnerable to attacks. Ensuring that only strong encryption algorithms are used increases security:
```bash
# Explicitly define strong encryption algorithms
Ciphers aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512,hmac-sha2-256
KexAlgorithms diffie-hellman-group-exchange-sha256
```

6) Enforce SSH idle timeout:

Inactive SSH sessions that are left open can be a security risk. Enforcing an idle timeout ensures that idle sessions are terminated after a set period, reducing exposure:
```bash
# These settings will automatically logout the idle user after 5 minutes
ClientAliveInterval 300 # Timeout after 5 minutes of inactivity
ClientAliveCountMax 0   # Disconnect the client after the timeout (no retries)
```

7) Enable Two-Factor Authentication (2FA) for SSH:

Adding two-factor authentication (2FA) to SSH enhances security by requiring a second form of identification (e.g. a OTP, smartcard, or app-based authenticator) in addition to an SSH key:
```bash
# Install and configure Google Authenticator or another PAM-baed 2FA solution:
sudo apt-get install libpam-google-authenticator -y 

# In /etc/pam.d/sshd, add the following line:
auth required pam_google_authenticator.so

# Enable challenge-response authentication
ChallengeResponseAuthentication yes
```

8) Limit User Login Attempts (Brute Force Protection):

Limiting the number of login attempts prevents attackers from repeatedly attempting passwords (or keys) in a brute-force attack. You can do this using Fail2Ban, which monitors SSH login attempts and automatically blocks IP addresses with too many failed login attempts:
```bash
# Installing fail2ban
sudo apt-get install fail2ban -y

# You can customize the failed login attempts for fail2ban in `/etc/fail2ban/jail.local`
[sshd]
enabled = true
port = 2222
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600 # Ban IP for 1 hour
```

9) Disabling unused SSH features:

These are all inside of the `/etc/sshd/sshd_config` configuration file:
```bash
# Disabling X11 Forwarding prevents attackers from using SSH to tunnel graphical        # applications:
X11Forwarding no

# Disabling TCP Forwarding stops users from forwarding network connections over SSH:
AllowTcpForwading no

# Disabling DNS lookups for incoming SSH connections helps speed up logins and prevents # DNS spoofing attacks
UseDNS no

# Disabling empty passwords ensures users cannot login without a password
PermitEmptyPasswords no

# Limits the number of authentication attempts per conenction to rpevent brute-force
MaxAuthRetries 3
```

10) Setting up SSH logging level to be verbose:

SSH logs are valuable for detecting and analyzing potential unauthorized access attempts. Increasing the logging level helps capture more information for auditing purposes:
```bash
# This logs more detailed information which is good for finding IoCs 
LogLevel VERBOSE
```

11) Enable SSH Banner (Optional - but it should not have revealing information):

A banner can warn unauthorized users that they are not allowed to access the system, which can serve as a deterrent, but banners can also expose information such as service versions (banner grabbing) which could be a vulnerability if present in the configuration file:
```bash
# Edit the SSH configuration file to include a file with a MOTD (Message Of The Day)
Banner /etc/issue.net

# You can add a warning message in this file like
Warning: Unauthorized access to this system is prohibited!
```

12) Disable SSH agent forwarding (if not needed):

SSH agent forwarding can be a security risk, if misused, as it allows remote servers to access your SSH keys. Disable this unless it is specifically needed:
```bash
AllowAgentForwarding no
```


13) Enforce a Login Grace Time:

This setting determines how long the SSH server will wait for the user to authenticate after a connection has been established. If a user fails to authenticate during this time, the session will be closed. This prevents attackers from holding open a connection indefinitely while they try different passwords or keys:
```bash
# Setting a time of 60 to mitigate DoS or brute-force attempts
LoginGraceTime 60
```

14) Enforcing the use of Protocol 2:

SSH supports two versions of the protocol, SSH-1 and SSH-2. SSH-1 is depreciated and contains numerous known security vulnerabilities. SSH-2 is more secure and should be enforced:
```bash
# Ensure your protocol parameter reflects 2
Protocol 2
```

15) Enforce the use of Privilege Separation:

This setting controls whether privilege separation is enabled for SSH or not. Privilege Separation helps mitigate security vulnerabilities by running different parts of the SSH process with different privilege levels (e.g. dropping root privileges when possible). It should be enabled to enhance security of the SSH service:
```bash
UsePrivilegeSeparation yes
```

16) Ignoring Rhosts:

This controls whether or not SSH will honor the `.rhosts` file for user authentication. The `.rhosts` file is a legacy (old) authentication mechanism from the old rlogin and rsh protocols that are insecure because they allow users to authenticate based on trusted hosts:
```bash
IgnoreRhosts yes
```

17) Host-Based Authentication:

Host-based authentication allows SSH to authenticate users based on the hostname of the client rather than requiring traditional user credentials. While this can be useful in trusted environments, it's generally considered insecure because it can be exploited if an attacker gains access to a trusted machine:
```bash
# If you're not using it, set the configuration
HostbasedAuthentication no
```

Attackers can spoof hostnames to gain unauthorized access, this solves for that.

18) Configure SSH with Restrict (For User-Specific Settings):

You can use the restrict option for users who need to be given limited access. This is useful for limiting what certain users can do after they login:
```bash
# Match certain commands or actions for specific users
Match User name_of_user_here
	ForceCommand /bin/false
	AllowTCPForwarding no
	X11Forwarding no
```

This configuration forces the user with the name of `name_of_user_here` to be unable to do anything but login, effectively preventing shell access or forwarding. Probably very scenario dependent. 
