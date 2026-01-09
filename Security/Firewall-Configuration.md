<div align="center">

# 🔥 Firewall Configuration Guide

**Complete OS-Level Firewall Setup for Maximum Security**

[![Firewall](https://img.shields.io/badge/Firewall-UFW%20%7C%20iptables-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-orange)]()

*Configure your server firewall to allow only essential traffic and block unnecessary ports*

---

</div>

## 📋 Overview

This guide will walk you through setting up a robust firewall configuration that:

- ✅ Implements OS-level firewall (UFW or iptables)
- ✅ Allows only SSL/TLS (443) and SSH (7022) ports
- ✅ Blocks all other incoming traffic by default
- ✅ Disables ICMP ping responses to prevent reconnaissance
- ✅ Maintains secure outbound connections

---

## 🚀 Step-by-Step Guide

### Step 1: Implement OS-Level Firewall

Choose your firewall solution based on your Linux distribution.

---

## 🛡️ Method 1: UFW (Uncomplicated Firewall) - Ubuntu/Debian

UFW is the default firewall management tool for Ubuntu and Debian systems.

### 1.1 Check UFW Status

```bash
sudo ufw status
```

### 1.2 Reset UFW to Default State (Optional)

If you have existing rules, reset to start fresh:

```bash
sudo ufw --force reset
```

### 1.3 Set Default Policies

Deny all incoming traffic by default, allow all outgoing:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 1.4 Allow Only Essential Ports

**Allow SSH on port 7022:**
```bash
sudo ufw allow 7022/tcp comment 'SSH access'
```

**Allow HTTPS on port 443:**
```bash
sudo ufw allow 443/tcp comment 'HTTPS traffic'
```

### 1.5 Enable UFW

```bash
sudo ufw enable
```

> ⚠️ **Warning:** Make sure you can access SSH on port 7022 before enabling, or you might lock yourself out!

### 1.6 Verify Configuration

```bash
# Check status and rules
sudo ufw status verbose

# Check numbered rules
sudo ufw status numbered

# Check logs
sudo ufw logging on
sudo tail -f /var/log/ufw.log
```

### 1.7 Disable ICMP Ping Responses (UFW)

UFW handles ICMP differently. Disable ping responses:

```bash
# Edit UFW configuration
sudo vi /etc/ufw/before.rules
```

Find the section that starts with `# ok icmp codes` and add before the `COMMIT` line:

```bash
# Disable ICMP ping
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
```

Or use a more complete ICMP block:

```bash
# Drop ICMP ping requests
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
-A ufw-before-input -p icmp --icmp-type echo-reply -j DROP
-A ufw-before-input -p icmp --icmp-type destination-unreachable -j DROP
```

Reload UFW:
```bash
sudo ufw reload
```

**Verify ICMP is blocked:**
```bash
# From another machine, try to ping your server
ping your-server-ip
# Should timeout or show no response
```

---

## 🔒 Method 2: iptables (Advanced/Legacy Systems)

iptables provides more granular control and works on all Linux distributions.

### 2.1 Install iptables-persistent (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install iptables-persistent -y
```

For CentOS/RHEL, iptables is usually pre-installed.

### 2.2 Flush Existing Rules (Optional - Start Fresh)

```bash
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t mangle -F
sudo iptables -t mangle -X
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

### 2.3 Set Default Policies

```bash
# Drop all incoming traffic by default
sudo iptables -P INPUT DROP

# Accept all outgoing traffic
sudo iptables -P OUTPUT ACCEPT

# Drop all forwarded traffic (if not routing)
sudo iptables -P FORWARD DROP
```

### 2.4 Allow Loopback Traffic

```bash
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
```

### 2.5 Allow Established and Related Connections

```bash
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

### 2.6 Allow SSH on Port 7022

```bash
sudo iptables -A INPUT -p tcp --dport 7022 -m state --state NEW -j ACCEPT
```

### 2.7 Allow HTTPS on Port 443

```bash
sudo iptables -A INPUT -p tcp --dport 443 -m state --state NEW -j ACCEPT
```

### 2.8 Disable ICMP Ping Responses

```bash
# Drop ICMP echo requests (ping)
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# Optionally, drop all ICMP traffic
# sudo iptables -A INPUT -p icmp -j DROP
```

### 2.9 Save iptables Rules

**For Ubuntu/Debian:**
```bash
sudo netfilter-persistent save
# Or manually:
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

**For CentOS/RHEL:**
```bash
sudo service iptables save
# Or:
sudo iptables-save > /etc/sysconfig/iptables
```

### 2.10 Verify iptables Configuration

```bash
# List all rules
sudo iptables -L -n -v

# List rules with line numbers
sudo iptables -L -n -v --line-numbers

# Check for specific port
sudo iptables -L -n | grep 7022
sudo iptables -L -n | grep 443
```

### 2.11 Test ICMP Blocking

```bash
# From another machine, test ping
ping your-server-ip
# Should timeout
```

---

## 🌐 Method 3: firewalld (CentOS/RHEL/Fedora)

firewalld is the default firewall manager for Red Hat-based systems.

### 3.1 Check firewalld Status

```bash
sudo systemctl status firewalld
```

### 3.2 Start and Enable firewalld

```bash
sudo systemctl start firewalld
sudo systemctl enable firewalld
```

### 3.3 Set Default Zone to drop (Most Restrictive)

```bash
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --reload
```

### 3.4 Remove Default Services

Remove default services to start clean:

```bash
sudo firewall-cmd --remove-service=ssh --permanent
sudo firewall-cmd --remove-service=dhcpv6-client --permanent
sudo firewall-cmd --reload
```

### 3.5 Allow SSH on Port 7022

```bash
sudo firewall-cmd --permanent --add-port=7022/tcp
sudo firewall-cmd --reload
```

### 3.6 Allow HTTPS on Port 443

```bash
sudo firewall-cmd --permanent --add-port=443/tcp
# Or add as service
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### 3.7 Disable ICMP Ping Responses

```bash
# Block ICMP echo requests
sudo firewall-cmd --permanent --add-icmp-block=echo-request
sudo firewall-cmd --reload

# Verify
sudo firewall-cmd --list-icmp-blocks
```

### 3.8 Verify firewalld Configuration

```bash
# Check active zones and rules
sudo firewall-cmd --list-all

# Check open ports
sudo firewall-cmd --list-ports

# Check services
sudo firewall-cmd --list-services

# Check ICMP blocks
sudo firewall-cmd --list-icmp-blocks
```

---

## 📝 Complete Configuration Examples

### UFW Complete Configuration

```bash
# Reset to defaults
sudo ufw --force reset

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow essential ports
sudo ufw allow 7022/tcp comment 'SSH'
sudo ufw allow 443/tcp comment 'HTTPS'

# Enable UFW
sudo ufw enable

# Configure ICMP blocking in /etc/ufw/before.rules
# Add: -A ufw-before-input -p icmp --icmp-type echo-request -j DROP

# Reload
sudo ufw reload
```

### iptables Complete Configuration Script

Create a script `/root/setup-iptables.sh`:

```bash
#!/bin/bash

# Flush existing rules
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (7022)
iptables -A INPUT -p tcp --dport 7022 -m state --state NEW -j ACCEPT

# Allow HTTPS (443)
iptables -A INPUT -p tcp --dport 443 -m state --state NEW -j ACCEPT

# Block ICMP ping
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# Save rules
iptables-save > /etc/iptables/rules.v4
```

Make executable and run:
```bash
sudo chmod +x /root/setup-iptables.sh
sudo /root/setup-iptables.sh
```

---

## ✅ Verification Checklist

After configuration, verify:

- [ ] Firewall is active and running
- [ ] SSH (7022) is accessible
- [ ] HTTPS (443) is accessible
- [ ] All other ports are blocked (try telnet on port 80: `telnet your-ip 80`)
- [ ] ICMP ping is blocked (no response from external machines)
- [ ] Outbound connections work (test: `curl google.com`)
- [ ] Firewall rules persist after reboot
- [ ] Firewall logging is enabled (if needed)

---

## 🔧 Troubleshooting

### Locked Out After Enabling Firewall

**If you can't access SSH:**

1. **Access via Console/KVM** (if available)
2. **Disable firewall temporarily:**
   - UFW: `sudo ufw disable`
   - iptables: `sudo iptables -P INPUT ACCEPT`
   - firewalld: `sudo systemctl stop firewalld`

### Cannot Access Web Application on Port 443

**Check if service is listening:**
```bash
sudo netstat -tlnp | grep 443
# Or
sudo ss -tlnp | grep 443
```

**Check firewall logs:**
```bash
# UFW
sudo tail -f /var/log/ufw.log

# iptables
sudo dmesg | grep DROP

# firewalld
sudo journalctl -u firewalld -f
```

### Need to Allow Additional Ports Temporarily

**UFW:**
```bash
sudo ufw allow 8080/tcp
# Remove later: sudo ufw delete allow 8080/tcp
```

**iptables:**
```bash
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4
```

**firewalld:**
```bash
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

### ICMP Still Responding

**Verify rules are active:**
```bash
# UFW - check before.rules
sudo grep -A5 "icmp" /etc/ufw/before.rules

# iptables - check rules
sudo iptables -L -n | grep icmp

# firewalld - check blocks
sudo firewall-cmd --list-icmp-blocks
```

**Test from another machine:**
```bash
ping your-server-ip
# Should timeout or show "Destination host unreachable"
```

---

## 🎯 Advanced Configuration

### Rate Limiting SSH Connections

**With iptables:**
```bash
# Limit SSH to 3 connections per minute
sudo iptables -A INPUT -p tcp --dport 7022 -m state --state NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 7022 -m state --state NEW -m recent --update --seconds 60 --hitcount 3 -j DROP
```

### Logging Dropped Packets

**iptables logging:**
```bash
sudo iptables -A INPUT -j LOG --log-prefix "DROPPED: " --log-level 4
sudo iptables -A INPUT -j DROP
```

**View logs:**
```bash
sudo tail -f /var/log/kern.log | grep DROPPED
```

### Whitelist Specific IP Addresses

**UFW:**
```bash
sudo ufw allow from 192.168.1.100 to any port 22
```

**iptables:**
```bash
sudo iptables -A INPUT -s 192.168.1.100 -p tcp --dport 22 -j ACCEPT
```

**firewalld:**
```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port port="22" protocol="tcp" accept'
sudo firewall-cmd --reload
```

---

## 📚 Security Best Practices

1. ✅ **Deny by default** - Block all incoming traffic, allow only what's needed
2. ✅ **Allow specific ports only** - 443 (HTTPS) and 7022 (SSH)
3. ✅ **Block ICMP ping** - Prevent network reconnaissance
4. ✅ **Use stateful rules** - Allow established connections
5. ✅ **Log firewall activity** - Monitor blocked attempts
6. ✅ **Test before production** - Verify configuration doesn't lock you out
7. ✅ **Keep firewall updated** - Regular security patches
8. ✅ **Document exceptions** - Track why ports are open

---

<div align="center">

**🔥 Your firewall is now configured for maximum security!**

[← Back to Handbook](../README.md)

</div>

