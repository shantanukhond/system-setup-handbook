<div align="center">

# 🚀 System Setup Handbook

**Your Complete Guide to System Configuration & Setup**

[![Documentation](https://img.shields.io/badge/Documentation-Updated-brightgreen)]()
[![License](https://img.shields.io/badge/License-MIT-blue)]()

*A comprehensive collection of guides, tutorials, and best practices for system setup and configuration.*

---

</div>

## 📑 Table of Contents

- [🔐 SSH Configuration](#-ssh-configuration)
  - [Organizing SSH Keys](./SSH/Organizing%20SSH%20Keys.md)
- [🔒 Security](#-security)
  - [SSH Hardening](./Security/SSH-Hardening.md)
  - [Firewall Configuration](./Security/Firewall-Configuration.md)
  - [Malware Protection](./Security/Malware-Protection.md)
  - [Cloudflare Security Setup](./Security/Cloudflare-Security-Setup.md)
- [⚡ Ease of Use](#-ease-of-use)
  - [Makefile Guide](./Productivity/Makefile-Guide.md)
  - [ZSH Configuration](./Productivity/ZSH-Configuration.md)

---

## 🔐 SSH Configuration

### 🔑 Organizing SSH Keys

Learn how to set up and manage SSH keys for virtual machines with a clean, organized configuration approach. This guide covers everything from generating keys to managing multiple server configurations efficiently.

**What you'll learn:**
- ✅ Organizing SSH keys by customer/project
- ✅ Configuring SSH config files for multiple hosts
- ✅ Generating and managing SSH keys securely
- ✅ Setting up seamless VM connections

**[📖 View Complete Guide →](./SSH/Organizing%20SSH%20Keys.md)**

---

## 🔒 Security

### 🛡️ SSH Hardening

Complete guide to hardening your SSH server against brute force attacks and unauthorized access. Learn essential security practices to protect your servers.

**What you'll learn:**
- ✅ Changing default SSH port (22 → 7022)
- ✅ Creating sudo users with SSH key authentication
- ✅ Disabling password authentication (key-only access)
- ✅ Implementing fail2ban for automatic IP banning
- ✅ Rate limiting SSH connections to prevent brute force

**[📖 View Complete Guide →](./Security/SSH-Hardening.md)**

---

### 🔥 Firewall Configuration

Comprehensive guide to configuring OS-level firewalls (UFW/iptables/firewalld) for maximum server security. Learn how to restrict access and protect your server.

**What you'll learn:**
- ✅ Implementing OS-level firewall (UFW/iptables/firewalld)
- ✅ Allowing only SSL/TLS (443) and SSH (7022) ports
- ✅ Blocking all unnecessary incoming traffic
- ✅ Disabling ICMP ping responses
- ✅ Configuring advanced firewall rules

**[📖 View Complete Guide →](./Security/Firewall-Configuration.md)**

---

### 🛡️ Malware Protection

Set up ClamAV antivirus to protect your Linux server from malware, trojans, and viruses. Configure automated scanning and threat detection.

**What you'll learn:**
- ✅ Installing and configuring ClamAV antivirus
- ✅ Updating virus definition databases
- ✅ Scheduling regular system scans
- ✅ Setting up email alerts for threats
- ✅ Optimizing scan performance

**[📖 View Complete Guide →](./Security/Malware-Protection.md)**

---

### ☁️ Cloudflare Security Setup

Complete guide to setting up Cloudflare proxy and origin certificates for encrypted server-to-Cloudflare communication and enhanced security.

**What you'll learn:**
- ✅ Enabling Cloudflare proxy for all subdomains
- ✅ Generating Origin Certificates for encryption
- ✅ Installing certificates on web servers (Nginx/Apache)
- ✅ Updating subdomain naming conventions (dots to dashes)
- ✅ Configuring SSL/TLS settings in Cloudflare

**[📖 View Complete Guide →](./Security/Cloudflare-Security-Setup.md)**

---

## ⚡ Ease of Use

### 🔧 Makefile Guide

Learn how to automate your development workflow with Makefiles. Create reusable commands, standardize operations, and save time with powerful automation.

**What you'll learn:**
- ✅ Creating and using Makefiles for task automation
- ✅ Defining custom commands and targets
- ✅ Managing variables and parameters
- ✅ Organizing complex workflows
- ✅ Server management and deployment automation

**[📖 View Complete Guide →](./Productivity/Makefile-Guide.md)**

---

### ⚡ ZSH Configuration

Transform your terminal into a powerful, beautiful, and efficient development environment with Zsh and Oh My Zsh.

**What you'll learn:**
- ✅ Installing and configuring Zsh (Z Shell)
- ✅ Setting up Oh My Zsh with plugins and themes
- ✅ Creating custom aliases and functions
- ✅ Configuring advanced features like autocompletion
- ✅ Customizing your terminal prompt and appearance

**[📖 View Complete Guide →](./Productivity/ZSH-Configuration.md)**

---

<div align="center">

**⭐ Star this repo if you find it helpful!**

*Made with ❤️ for developers*

</div>
