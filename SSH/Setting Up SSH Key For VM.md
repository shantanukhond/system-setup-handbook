<div align="center">

# 🔐 Setting Up SSH Key for VM

**Organized SSH Configuration Management Guide**

[![SSH](https://img.shields.io/badge/SSH-Configuration-blue)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-orange)]()

*Learn how to set up and manage SSH keys for virtual machines with a clean, organized approach*

---

</div>

## 📋 Overview

This guide will walk you through setting up SSH keys for VMs using an organized configuration structure. This approach allows you to:

- ✅ Manage multiple servers and customers efficiently
- ✅ Keep SSH keys organized by project/customer
- ✅ Use a modular configuration system
- ✅ Easily add or remove server configurations

---

## 🚀 Step-by-Step Guide

### Step 1: Configure Main SSH Config File

Edit your main SSH config file and add the include directive at the top:

```bash
vi ~/.ssh/config
```

Add this line at the **top** of the file:

```bash
Include ~/.ssh/_config/*.config
```

> 💡 **Tip:** This allows you to split your SSH configurations into multiple files, keeping them organized and manageable.

---

### Step 2: Create Configuration Directory

Create the `_config` directory in your `~/.ssh` folder:

```bash
mkdir ~/.ssh/_config
```

> 📝 **Note:** You can also use `md ~/.ssh/_config` if you prefer.

---

### Step 3: Create Customer/Project Configuration File

Create a configuration file for each customer or project. For example:

```bash
vi ~/.ssh/_config/cus.config
```

Add the following configuration template and update it according to your needs:

```bash
Host sub.example.com
  HostName ip-here
  IdentityFile ~/.ssh/cus/sub.example.com
  User ubuntu
  IdentitiesOnly yes
  SetEnv SSH_ALIAS_NAME=sub.example.com
```

**Configuration Breakdown:**
- `Host` - The alias you'll use to connect (e.g., `ssh sub.example.com`)
- `HostName` - The actual IP address or domain of your server
- `IdentityFile` - Path to your private SSH key
- `User` - Username on the remote server (e.g., `ubuntu`, `root`, `ec2-user`)
- `IdentitiesOnly` - Ensures only the specified key is used
- `SetEnv` - Sets an environment variable (optional, for reference)

---

### Step 4: Organize Your Folder Structure

Create a folder structure to organize your SSH keys. For example:

```bash
mkdir -p ~/.ssh/cus
```

You can create separate directories for different customers or projects:
- `~/.ssh/cus/` - Customer keys
- `~/.ssh/projects/` - Project-specific keys
- `~/.ssh/personal/` - Personal keys

---

### Step 5: Generate SSH Key

Generate a new SSH key using the ED25519 algorithm (recommended):

```bash
ssh-keygen -t ed25519 -C youremail@example.com
```

**During the key generation:**
- When prompted for the file name, use the same name as your `IdentityFile` (e.g., `sub.example.com`)
- You can set a passphrase for extra security (recommended)
- After creation, move the key to the appropriate folder if needed

**Example:**
```bash
# If you saved it as ~/.ssh/sub.example.com
mv ~/.ssh/sub.example.com ~/.ssh/cus/sub.example.com
mv ~/.ssh/sub.example.com.pub ~/.ssh/cus/sub.example.com.pub
```

---

### Step 6: Add Public Key to Server

Copy your public key to the server's `authorized_keys` file:

**Option A: Using `ssh-copy-id` (Easier)**
```bash
ssh-copy-id -i ~/.ssh/cus/sub.example.com.pub ubuntu@ip-here
```

**Option B: Manual Method**
1. Copy your public key:
   ```bash
   cat ~/.ssh/cus/sub.example.com.pub
   ```

2. SSH into your server (using password):
   ```bash
   ssh ubuntu@ip-here
   ```

3. Edit the authorized_keys file:
   ```bash
   vi ~/.ssh/authorized_keys
   ```

4. Paste your public key and save the file

---

### Step 7: Test Your Connection

Test if you can connect to your VM using the configured alias:

```bash
ssh sub.example.com
```

If everything is set up correctly, you should connect without entering a password! 🎉

---

## 📝 Complete Configuration Example

Here's a complete example of what your configuration file might look like:

```bash
Host sub.example.com
  HostName 192.168.1.100
  IdentityFile ~/.ssh/cus/sub.example.com
  User ubuntu
  IdentitiesOnly yes
  SetEnv SSH_ALIAS_NAME=sub.example.com
```

---

## ✅ Verification Checklist

- [ ] Main SSH config includes `~/.ssh/_config/*.config`
- [ ] `_config` directory exists in `~/.ssh/`
- [ ] Customer/project config file created with correct settings
- [ ] SSH key generated and placed in correct location
- [ ] Public key added to server's `authorized_keys`
- [ ] Connection tested successfully

---

## 🔧 Troubleshooting

**Issue: Connection still asks for password**
- Verify the public key is in `~/.ssh/authorized_keys` on the server
- Check file permissions: `chmod 600 ~/.ssh/authorized_keys` on server
- Ensure `IdentityFile` path in config matches your actual key location

**Issue: "Permission denied (publickey)"**
- Verify the private key permissions: `chmod 600 ~/.ssh/cus/sub.example.com`
- Check that `IdentitiesOnly yes` is set in your config
- Ensure the key name matches in both config and actual file

---

<div align="center">

**🎉 You're all set! Happy SSH-ing!**

[← Back to Handbook](../README.md)

</div>
