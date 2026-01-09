<div align="center">

# 🔒 SSH Hardening Guide

**Complete Security Hardening for SSH Servers**

[![Security](https://img.shields.io/badge/Security-Hardening-red)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-orange)]()

*Protect your SSH server from brute force attacks and unauthorized access*

---

</div>

## 📋 Overview

This guide will walk you through essential SSH hardening techniques to secure your server. You'll learn how to:

- ✅ Change the default SSH port (reduce automated attacks)
- ✅ Create a sudo user with SSH key authentication
- ✅ Disable password authentication (SSH key-only)
- ✅ Implement fail2ban for automatic IP banning
- ✅ Implement rate limiting to prevent brute force attacks

> ⚠️ **Important:** Follow these steps carefully and keep an active SSH session open until you verify everything works. Locking yourself out of your server is a real risk!

---

## 🚀 Step-by-Step Guide

### Step 1: Change Default SSH Port from 22 to 7022

Changing the default port reduces automated attacks from bots scanning port 22.

#### 1.1 Edit SSH Configuration

```bash
sudo vi /etc/ssh/sshd_config
```

#### 1.2 Find and Modify the Port Line

Find this line:
```bash
#Port 22
```

Change it to:
```bash
Port 7022
```

> 💡 **Tip:** Remove the `#` comment symbol to activate the setting.

#### 1.3 Update Firewall Rules

**For UFW (Ubuntu Firewall):**
```bash
sudo ufw allow 7022/tcp
sudo ufw delete allow 22/tcp  # Remove old port if needed
sudo ufw reload
```

**For firewalld (CentOS/RHEL):**
```bash
sudo firewall-cmd --permanent --add-port=7022/tcp
sudo firewall-cmd --remove-service=ssh  # Remove old SSH service
sudo firewall-cmd --reload
```

**For iptables:**
```bash
sudo iptables -A INPUT -p tcp --dport 7022 -j ACCEPT
sudo iptables-save
```

#### 1.4 Test New Port (Before Restarting SSH)

**Keep your current session open!** Test the new port in a new terminal:

```bash
ssh -p 7022 username@your-server-ip
```

#### 1.5 Restart SSH Service

```bash
sudo systemctl restart sshd
# Or on some systems:
sudo systemctl restart ssh
```

#### 1.6 Update Your SSH Config

Update your local `~/.ssh/config` file:

```bash
Host your-server-alias
  HostName your-server-ip
  Port 7022
  User your-username
  IdentityFile ~/.ssh/your-key
```

---

### Step 2: Create Sudo User with SSH Key Authentication

Never use root for SSH access. Create a sudo user instead.

#### 2.1 Create New User

```bash
sudo adduser newusername
# Follow the prompts to set password and user info
```

Or for non-interactive creation:

```bash
sudo useradd -m -s /bin/bash newusername
sudo passwd newusername
```

#### 2.2 Add User to Sudo Group

**For Ubuntu/Debian:**
```bash
sudo usermod -aG sudo newusername
```

**For CentOS/RHEL:**
```bash
sudo usermod -aG wheel newusername
```

#### 2.3 Generate SSH Key on Your Local Machine

On your local computer, generate an SSH key:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/newusername_server_key
```

#### 2.4 Copy Public Key to New User on Server

**Method 1: Using ssh-copy-id (Recommended)**
```bash
ssh-copy-id -i ~/.ssh/newusername_server_key.pub -p 7022 newusername@your-server-ip
```

**Method 2: Manual Method**
```bash
# On your local machine, display the public key
cat ~/.ssh/newusername_server_key.pub

# SSH to server as the new user (use password initially)
ssh -p 7022 newusername@your-server-ip

# On the server, create .ssh directory if it doesn't exist
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add the public key to authorized_keys
echo "YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

#### 2.5 Test Sudo Access

```bash
# Test sudo access
sudo whoami
# Should output: root

# Test with a command
sudo ls /root
```

---

### Step 3: Disable Password Authentication (SSH Key-Only)

Disable password authentication to prevent brute force attacks.

#### 3.1 Edit SSH Configuration

```bash
sudo vi /etc/ssh/sshd_config
```

#### 3.2 Configure Authentication Settings

Find and modify these lines:

```bash
# Disable password authentication
PasswordAuthentication no
PubkeyAuthentication yes

# Disable root login (if not already disabled)
PermitRootLogin no

# Optional: Specify which users can login
AllowUsers newusername

# Optional: Disable empty passwords
PermitEmptyPasswords no
```

#### 3.3 Verify Key Authentication Works First!

**Before restarting SSH, ensure your key authentication works:**

```bash
# Test from local machine
ssh -i ~/.ssh/newusername_server_key -p 7022 newusername@your-server-ip
```

#### 3.4 Restart SSH Service

```bash
sudo systemctl restart sshd
```

#### 3.5 Verify Configuration

Check that password auth is disabled:

```bash
sudo sshd -T | grep passwordauthentication
# Should output: passwordauthentication no
```

---

### Step 4: Implement fail2ban to Automatically Ban IPs

fail2ban monitors log files and bans IPs showing malicious activity.

#### 4.1 Install fail2ban

**For Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install fail2ban -y
```

**For CentOS/RHEL:**
```bash
sudo yum install epel-release -y
sudo yum install fail2ban -y
```

**For Fedora:**
```bash
sudo dnf install fail2ban -y
```

#### 4.2 Create Local Configuration File

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vi /etc/fail2ban/jail.local
```

#### 4.3 Configure SSH Jail

Find the `[sshd]` section and configure it:

```bash
[sshd]
enabled = true
port = 7022  # Match your new SSH port
filter = sshd
logpath = /var/log/auth.log  # For Debian/Ubuntu
# logpath = /var/log/secure  # For CentOS/RHEL
maxretry = 3
bantime = 3600
findtime = 600
```

**Configuration Explanation:**
- `maxretry = 3` - Ban after 3 failed attempts
- `bantime = 3600` - Ban for 1 hour (3600 seconds)
- `findtime = 600` - Look back 10 minutes (600 seconds)

#### 4.4 Whitelist Your Static IP Address

To prevent your static IP from ever being banned, add it to the ignore list:

**Method 1: Add to Jail Configuration (Recommended)**

Edit the SSH jail configuration:

```bash
sudo vi /etc/fail2ban/jail.local
```

Find the `[sshd]` section and add the `ignoreip` line:

```bash
[sshd]
enabled = true
port = 7022
filter = sshd
logpath = /var/log/auth.log  # For Debian/Ubuntu
# logpath = /var/log/secure  # For CentOS/RHEL
maxretry = 3
bantime = 3600
findtime = 600
ignoreip = 127.0.0.1/8 ::1 YOUR_STATIC_IP_ADDRESS
```

**For multiple IPs:**
```bash
ignoreip = 127.0.0.1/8 ::1 192.168.1.100 203.0.113.45 198.51.100.25
```

**Method 2: Add to Global Default Section**

You can also add it to the `[DEFAULT]` section at the top of the file to apply to all jails:

```bash
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 YOUR_STATIC_IP_ADDRESS
```

**Method 3: Create a Whitelist File**

For better organization, create a separate whitelist file:

```bash
sudo vi /etc/fail2ban/ignoreip.conf
```

Add your IPs:
```
127.0.0.1/8
::1
YOUR_STATIC_IP_ADDRESS
```

Then include it in `/etc/fail2ban/jail.local`:
```bash
[DEFAULT]
# Include whitelist from external file
ignoreip = /etc/fail2ban/ignoreip.conf
```

**Restart fail2ban after making changes:**
```bash
sudo systemctl restart fail2ban
```

**Verify whitelist is active:**
```bash
# Check configuration
sudo fail2ban-client get sshd ignoreip

# Test that your IP is whitelisted (should not show in banned list)
sudo fail2ban-client status sshd
```

> 💡 **Tip:** The `127.0.0.1/8` and `::1` are default whitelist entries for localhost. Always keep them unless you have a specific reason to remove them.

#### 4.5 For Custom SSH Port, Update Filter

Edit the SSH filter to recognize your custom port:

```bash
sudo vi /etc/fail2ban/filter.d/sshd.conf
```

Add or modify the port regex (if needed):
```bash
port\s+=\s+(?P<port>\d+)
```

#### 4.6 Update logpath Based on Your System

**Check which log file your system uses:**

```bash
# Debian/Ubuntu
sudo tail -f /var/log/auth.log

# CentOS/RHEL/Fedora
sudo tail -f /var/log/secure
```

#### 4.7 Start and Enable fail2ban

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

#### 4.8 Check fail2ban Status

```bash
# Check status
sudo systemctl status fail2ban

# Check active jails
sudo fail2ban-client status

# Check SSH jail specifically
sudo fail2ban-client status sshd

# View banned IPs
sudo fail2ban-client status sshd | grep Banned

# Verify your IP is whitelisted (should not appear in banned list)
sudo fail2ban-client get sshd ignoreip
```

#### 4.9 Test fail2ban (Optional)

**⚠️ Warning: This will ban your IP!**

```bash
# Try to SSH with wrong password 3+ times
ssh -p 7022 wronguser@your-server-ip

# Check if IP is banned
sudo fail2ban-client status sshd

# Unban your IP if needed
sudo fail2ban-client set sshd unbanip YOUR_IP_ADDRESS
```

---

### Step 5: Implement Rate Limiting on SSH Port

Rate limiting adds an extra layer of protection against brute force attacks.

#### 5.1 Using iptables (Method 1 - Recommended)

**Install iptables-persistent (Debian/Ubuntu):**
```bash
sudo apt install iptables-persistent -y
```

**Add rate limiting rules:**
```bash
# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Rate limit new SSH connections (5 connections per minute)
sudo iptables -A INPUT -p tcp --dport 7022 -m state --state NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 7022 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j DROP

# Allow SSH on port 7022
sudo iptables -A INPUT -p tcp --dport 7022 -j ACCEPT

# Save rules (Ubuntu/Debian)
sudo netfilter-persistent save
# Or
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

**For CentOS/RHEL:**
```bash
# Save iptables rules
sudo service iptables save
# Or
sudo iptables-save > /etc/sysconfig/iptables
```

#### 5.2 Using UFW with Rate Limiting (Method 2 - Ubuntu)

**UFW doesn't have built-in rate limiting, but you can use iptables with UFW:**

```bash
# Edit UFW before rules
sudo vi /etc/ufw/before.rules
```

Add before the `*filter` section:
```bash
# Rate limiting for SSH
-A ufw-before-input -p tcp --dport 7022 -m state --state NEW -m recent --set
-A ufw-before-input -p tcp --dport 7022 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j DROP
```

#### 5.3 Using firewalld with rich rules (Method 3 - CentOS/RHEL)

```bash
# Add rate limiting rule
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" service name="ssh" limit value=5/m accept'
sudo firewall-cmd --reload
```

**For custom port:**
```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" port port=7022 protocol=tcp limit value=5/m accept'
sudo firewall-cmd --reload
```

#### 5.4 Verify Rate Limiting

Test by making multiple rapid connections:

```bash
# On your local machine, try rapid connections
for i in {1..10}; do ssh -p 7022 newusername@your-server-ip & done

# Check if connections are being dropped
# You should see some connections succeed and others fail
```

Check firewall logs:
```bash
# For iptables
sudo tail -f /var/log/kern.log | grep DROP
# Or
sudo dmesg | grep DROP
```

---

## ✅ Complete Configuration Summary

### Final `/etc/ssh/sshd_config` Snippets

```bash
# Change default port
Port 7022

# Disable password authentication
PasswordAuthentication no
PubkeyAuthentication yes

# Disable root login
PermitRootLogin no

# Allow specific users
AllowUsers newusername

# Security settings
PermitEmptyPasswords no
X11Forwarding no  # Disable if not needed
```

### Final fail2ban Configuration Snippets

```bash
[sshd]
enabled = true
port = 7022
filter = sshd
logpath = /var/log/auth.log  # Or /var/log/secure for CentOS
maxretry = 3
bantime = 3600
findtime = 600
ignoreip = 127.0.0.1/8 ::1 YOUR_STATIC_IP_ADDRESS  # Whitelist your static IP
action = %(action_)s
```

### Final iptables Rate Limiting Snippets

```bash
# Rate limit: 5 new connections per minute
-A INPUT -p tcp --dport 7022 -m state --state NEW -m recent --set
-A INPUT -p tcp --dport 7022 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j DROP
```

---

## 🔍 Verification Checklist

After completing all steps, verify:

- [ ] SSH port changed to 7022 and accessible
- [ ] Sudo user created and can use sudo
- [ ] SSH key authentication works for new user
- [ ] Password authentication is disabled
- [ ] Root login is disabled
- [ ] Static IP whitelisted in fail2ban (never gets banned)
- [ ] fail2ban is running and monitoring
- [ ] Rate limiting rules are active
- [ ] Can still SSH into server successfully
- [ ] Firewall rules are persistent after reboot

---

## 🔧 Troubleshooting

### Cannot Connect After Changes

**If you're locked out:**

1. Use server console/KVM access if available
2. Temporarily revert changes via console
3. Check firewall isn't blocking you
4. Verify SSH service is running: `sudo systemctl status sshd`

### fail2ban Not Working

```bash
# Check fail2ban logs
sudo tail -f /var/log/fail2ban.log

# Verify jail is active
sudo fail2ban-client status sshd

# Test filter
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf
```

### Rate Limiting Too Strict

Adjust the hitcount in iptables rules:
```bash
# Change --hitcount 5 to a higher number like 10
sudo vi /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
```

### Port Not Accessible

```bash
# Check if port is listening
sudo netstat -tlnp | grep 7022
# Or
sudo ss -tlnp | grep 7022

# Check firewall
sudo ufw status
# Or
sudo firewall-cmd --list-all
```

---

## 📝 Security Best Practices Summary

1. ✅ **Always use non-default ports** - Reduces automated attacks
2. ✅ **Never use root for SSH** - Create sudo users instead
3. ✅ **Disable password auth** - Use SSH keys only
4. ✅ **Monitor and ban attackers** - Use fail2ban
5. ✅ **Rate limit connections** - Prevent brute force attacks
6. ✅ **Keep SSH updated** - Regular security patches
7. ✅ **Use strong SSH keys** - ED25519 or RSA 4096-bit
8. ✅ **Restrict user access** - Use `AllowUsers` or `AllowGroups`

---

<div align="center">

**🔒 Your SSH server is now significantly more secure!**

[← Back to Handbook](../README.md)

</div>

