<div align="center">

# ☁️ Cloudflare Security Setup Guide

**Complete Cloudflare Proxy and Origin Certificate Configuration**

[![Cloudflare](https://img.shields.io/badge/Cloudflare-Proxy-orange)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-orange)]()

*Enable Cloudflare proxy, generate origin certificates, and secure server-to-Cloudflare connections*

---

</div>

## 📋 Overview

This guide will walk you through setting up Cloudflare security features:

- ✅ Enable Cloudflare proxy (orange cloud) for all subdomains
- ✅ Generate Origin Certificate for encrypted server-to-Cloudflare communication
- ✅ Install and configure Origin Certificate on your server
- ✅ Update subdomain naming: convert dots (.) to dashes (-) for certificate compatibility
- ✅ Configure web server (Nginx/Apache) with Origin Certificate

> 💡 **Note:** Cloudflare Origin Certificates are free SSL certificates that encrypt traffic between Cloudflare and your origin server, even if your origin server doesn't have a public IP or valid domain certificate.

**Reference:** For more information about Cloudflare Origin CA, see the [official Cloudflare blog post on Origin CA encryption](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/).

---

## 🚀 Step-by-Step Guide

### Step 1: Enable Cloudflare Proxy for Subdomains

#### 1.1 Access Cloudflare Dashboard

1. Log in to your [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Select your domain

#### 1.2 Navigate to DNS Settings

1. Click on **DNS** in the left sidebar
2. You'll see a list of all DNS records

#### 1.3 Enable Proxy for Subdomains

For each subdomain you want to proxy:

1. Find the A or CNAME record for your subdomain
2. Click on the **orange cloud icon** (☁️) to enable proxy
   - **Orange cloud** = Proxied (through Cloudflare)
   - **Grey cloud** = DNS only (direct to origin)

#### 1.4 Enable Proxy for All Subdomains (Bulk Method)

If you have many subdomains, use Cloudflare API or bulk edit:

**Via Cloudflare Dashboard:**
1. In DNS settings, select multiple records using checkboxes
2. Use bulk actions to toggle proxy status

**Via Cloudflare API:**
```bash
# List all DNS records
curl -X GET "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json"

# Update record to proxy (set proxied: true)
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records/{record_id}" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"proxied":true}'
```

#### 1.5 Verify Proxy Status

Check that all subdomains show the orange cloud icon in the DNS records list.

---

### Step 2: Generate Origin Certificate

#### 2.1 Navigate to SSL/TLS Settings

1. In Cloudflare Dashboard, go to **SSL/TLS**
2. Click on **Origin Server** tab

#### 2.2 Create Origin Certificate

1. Click **Create Certificate** button
2. Configure certificate settings:

**Certificate Settings:**

```bash
# Private key type (recommended: RSA 2048 or ECC)
Private key type: RSA (2048) or ECDSA P-256

# Hostnames (add all subdomains and domains)
Hostnames:
  - example.com
  - *.example.com
  - sub1.example.com
  - sub2.example.com
  - api.example.com

# Certificate Validity
Validity: 15 years (maximum)

# Private Key Format (for web server)
Format: Origin Certificate + Private Key
```

**Important Notes:**
- Use `*.example.com` for wildcard subdomains
- List specific subdomains explicitly if needed
- Certificate is valid for 15 years (Cloudflare managed)

#### 2.3 Download Certificate and Private Key

After creating the certificate:

1. **Copy the Origin Certificate** (public certificate)
2. **Copy the Private Key** (keep this secret!)

> ⚠️ **Warning:** Save the private key immediately! Cloudflare won't show it again after you close this dialog.

#### 2.4 Save Certificate Files

Save the certificate and key to secure files:

```bash
# Create directory for certificates
sudo mkdir -p /etc/ssl/cloudflare

# Save Origin Certificate (public)
sudo vi /etc/ssl/cloudflare/origin.crt
# Paste the certificate content here

# Save Private Key
sudo vi /etc/ssl/cloudflare/origin.key
# Paste the private key content here

# Set proper permissions
sudo chmod 600 /etc/ssl/cloudflare/origin.key
sudo chmod 644 /etc/ssl/cloudflare/origin.crt
sudo chown root:root /etc/ssl/cloudflare/*
```

---

### Step 3: Update Subdomain Naming Convention

Cloudflare Origin Certificates require consistent naming. If your subdomains use dots (`.`), you may need to convert them to dashes (`-`) for compatibility.

#### 3.1 Understanding the Issue

**Problem:** Some certificate authorities and Cloudflare may have issues with:
- Multiple consecutive dots
- Dots in certain positions
- Wildcards with complex dot patterns

**Solution:** Use dashes instead of dots for subdomain separators.

#### 3.2 Example Conversions

```bash
# Before (with dots)
api.subdomain.example.com
app.sub.domain.example.com
user.api.example.com

# After (with dashes)
api-subdomain.example.com
app-sub-domain.example.com
user-api.example.com
```

#### 3.3 Update DNS Records in Cloudflare

1. **Delete old DNS records** (with dots)
2. **Create new DNS records** (with dashes)

**Example:**
```bash
# Old record
Type: A
Name: api.subdomain
Content: 192.168.1.100
Proxy: ON

# New record
Type: A
Name: api-subdomain
Content: 192.168.1.100
Proxy: ON
```

#### 3.4 Update Origin Certificate

Update the certificate hostnames to match new naming:

1. Go to **SSL/TLS** → **Origin Server**
2. Edit existing certificate or create new one
3. Update hostnames to use dashes:

```bash
Hostnames:
  - example.com
  - *.example.com
  - api-subdomain.example.com
  - app-sub-domain.example.com
  - user-api.example.com
```

#### 3.5 Update Application Configuration

Update your application configurations to use new subdomain names:

```bash
# Update environment variables
API_URL=https://api-subdomain.example.com
APP_URL=https://app-sub-domain.example.com

# Update database records (if storing domains)
UPDATE domains SET subdomain = 'api-subdomain' WHERE subdomain = 'api.subdomain';
```

#### 3.6 Verify DNS Propagation

```bash
# Check DNS records
dig api-subdomain.example.com
dig api-subdomain.example.com +short

# Check with Cloudflare proxy
curl -I https://api-subdomain.example.com
```

---

### Step 4: Install Origin Certificate on Server

#### 4.1 Copy Certificate Files to Server

If you saved certificates locally, copy to server:

```bash
# Using SCP
scp -P 7022 origin.crt user@server:/tmp/
scp -P 7022 origin.key user@server:/tmp/

# On server, move to secure location
sudo mv /tmp/origin.crt /etc/ssl/cloudflare/
sudo mv /tmp/origin.key /etc/ssl/cloudflare/
sudo chmod 600 /etc/ssl/cloudflare/origin.key
sudo chmod 644 /etc/ssl/cloudflare/origin.crt
```

#### 4.2 Verify Certificate

```bash
# View certificate details
openssl x509 -in /etc/ssl/cloudflare/origin.crt -text -noout

# Check certificate validity
openssl x509 -in /etc/ssl/cloudflare/origin.crt -dates -noout

# Verify private key matches certificate
openssl x509 -noout -modulus -in /etc/ssl/cloudflare/origin.crt | openssl md5
openssl rsa -noout -modulus -in /etc/ssl/cloudflare/origin.key | openssl md5
# Both should output the same hash
```

---

### Step 5: Configure Nginx with Origin Certificate

#### 5.1 Update Nginx Configuration

Edit your Nginx server block:

```bash
sudo vi /etc/nginx/sites-available/example.com
```

**Add SSL configuration:**

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com *.example.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com *.example.com;

    # Cloudflare Origin Certificate
    ssl_certificate /etc/ssl/cloudflare/origin.crt;
    ssl_certificate_key /etc/ssl/cloudflare/origin.key;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Root directory
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### 5.2 Restart Nginx

```bash
# Test configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx

# Check status
sudo systemctl status nginx
```


---

### Step 7: Configure Cloudflare SSL/TLS Mode

#### 7.1 Set SSL/TLS Mode

In Cloudflare Dashboard:

1. Go to **SSL/TLS** → **Overview**
2. Set SSL/TLS encryption mode to **Full (strict)**

**Available modes:**
- **Off** - No encryption between Cloudflare and origin
- **Flexible** - Encrypts browser to Cloudflare only
- **Full** - Encrypts end-to-end (allows self-signed certificates)
- **Full (strict)** - Encrypts end-to-end (requires valid certificate) ✅ **Recommended**

**Full (strict) Mode Diagram:**

The diagram below illustrates how "Full (strict)" SSL/TLS encryption mode works:

![Cloudflare Full (strict) SSL/TLS Mode](https://blog.cloudflare.com/content/images/2016/05/ssl-encryption-mode-full-strict.png)

*Diagram showing end-to-end encryption with Cloudflare Origin Certificates - Source: [Cloudflare Blog](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/)*

**Text representation:**

```
┌─────────┐         Encrypted         ┌──────────┐         Encrypted         ┌─────────────┐
│ Visitor │◄─────────────────────────►│Cloudflare│◄─────────────────────────►│Origin Server│
└─────────┘    (Browser-trusted       └──────────┘   (Cloudflare-issued      └─────────────┘
                certificate)                          certificate)
                    │                                        │
                    ▼                                        ▼
         ┌──────────────────────┐              ┌──────────────────────┐
         │ Browser-trusted      │              │ Cloudflare-issued    │
         │ certificate          │              │ certificate          │
         └──────────────────────┘              └──────────────────────┘
```

In **Full (strict)** mode:
- **Visitor to Cloudflare**: Encrypted with browser-trusted certificate (presented by Cloudflare)
- **Cloudflare to Origin Server**: Encrypted with Cloudflare-issued Origin Certificate (on your server)
- **All segments** of the connection are encrypted end-to-end

#### 7.2 Enable Always Use HTTPS

1. Go to **SSL/TLS** → **Edge Certificates**
2. Enable **Always Use HTTPS**
3. Enable **Automatic HTTPS Rewrites**

#### 7.3 Configure Minimum TLS Version

1. Go to **SSL/TLS** → **Edge Certificates**
2. Set **Minimum TLS Version** to **TLS 1.2** or higher

---

## ✅ Verification Checklist

After setup, verify:

- [ ] All subdomains show orange cloud (proxied) in DNS
- [ ] Origin Certificate is generated and downloaded
- [ ] Certificate and key files are saved securely on server
- [ ] Subdomain names updated (dots to dashes if needed)
- [ ] DNS records updated in Cloudflare
- [ ] Origin Certificate installed on web server
- [ ] Web server configured with SSL
- [ ] HTTPS working on all subdomains
- [ ] SSL/TLS mode set to "Full (strict)"
- [ ] Always Use HTTPS enabled
- [ ] Test SSL connection: `curl -I https://your-subdomain.example.com`

---

## 🔧 Troubleshooting

### Certificate Not Trusted by Browser

**This is expected!** Origin Certificates are only trusted by Cloudflare, not browsers. Users will see valid SSL through Cloudflare's edge certificates.

**Verify from Cloudflare:**
```bash
curl -I https://your-subdomain.example.com
# Should show Cloudflare's SSL certificate
```

### 526 Error: Invalid SSL Certificate

**Problem:** Cloudflare can't validate your origin certificate.

**Solutions:**
1. Check SSL/TLS mode is set to **Full** or **Full (strict)**
2. Verify certificate is installed correctly on origin server
3. Check certificate hostnames match your domain
4. Verify certificate hasn't expired
5. Check web server is listening on port 443

```bash
# Test origin server SSL
openssl s_client -connect your-server-ip:443 -servername your-domain.com
```

### Subdomain Not Resolving After Name Change

**Check DNS propagation:**
```bash
dig new-subdomain-name.example.com
nslookup new-subdomain-name.example.com
```

**Clear DNS cache:**
```bash
# On your local machine
sudo systemd-resolve --flush-caches
# Or
sudo dscacheutil -flushcache
```

### Wildcard Certificate Not Working

**Problem:** `*.example.com` doesn't cover all subdomains.

**Solution:** List specific subdomains in certificate:

```bash
Hostnames in certificate:
  - example.com
  - *.example.com
  - specific-sub.example.com
  - api-sub.example.com
```

### Origin Certificate Expired

Origin Certificates are valid for 15 years. If expired:

1. Generate a new certificate in Cloudflare Dashboard
2. Download new certificate and key
3. Replace files on server:
```bash
sudo cp new-origin.crt /etc/ssl/cloudflare/origin.crt
sudo cp new-origin.key /etc/ssl/cloudflare/origin.key
sudo chmod 600 /etc/ssl/cloudflare/origin.key
```
4. Restart web server

---

## 📚 Best Practices

1. ✅ **Use Full (strict) mode** - Maximum security with valid certificates
2. ✅ **Enable Always Use HTTPS** - Automatic redirects
3. ✅ **Use consistent naming** - Dashes instead of dots for subdomains
4. ✅ **Keep private key secure** - Proper permissions (600) and secure storage
5. ✅ **Regular certificate rotation** - Even with 15-year validity
6. ✅ **Monitor SSL/TLS errors** - Check Cloudflare Analytics
7. ✅ **Test all subdomains** - Verify SSL works on each
8. ✅ **Backup certificates** - Store securely off-server

---

## 🔐 Security Considerations

- **Origin Certificates are free** but only trusted by Cloudflare
- **Private keys must be kept secret** - Never commit to version control
- **Use Full (strict) mode** for maximum security
- **Origin IP should be protected** - Use firewall to allow only Cloudflare IPs
- **Enable Authenticated Origin Pulls** - Additional security layer (optional)

---

## 📖 References

- [**Cloudflare Origin CA - Official Blog Post**](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/) - Learn more about Cloudflare Origin Certificates, their benefits, and how they differ from public certificates.

---

<div align="center">

**☁️ Your Cloudflare security setup is now complete!**

[← Back to Handbook](../README.md)

</div>

