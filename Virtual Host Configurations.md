### `lab5.conf` — HTTP Redirect Only
```apache
<VirtualHost *:80>
    ServerAdmin mridul@lab5.webpentesting
    ServerName lab5.webpentesting
    Redirect permanent / https://lab5.webpentesting/
</VirtualHost>
```

---

### `lab5-ssl.conf` — HTTPS Main Site
```apache
<VirtualHost *:443>
    ServerAdmin mridul@lab5.webpentesting
    ServerName lab5.webpentesting
    DocumentRoot /var/www/WebSecurityEnvironment

    SSLEngine on
    SSLCertificateFile    /etc/ssl/localcerts/mridul.pem
    SSLCertificateKeyFile /etc/ssl/localcerts/mridul.key

    <Directory /var/www/WebSecurityEnvironment>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/lab5-error.log
    CustomLog ${APACHE_LOG_DIR}/lab5-access.log combined
</VirtualHost>
```

---

### `lab5-2.conf` — Second Site (Different Background)
```apache
<VirtualHost *:80>
    ServerAdmin mridul@lab5-2.webpentesting
    ServerName lab5-2.webpentesting
    DocumentRoot /var/www/WebSecurityEnvironment2

    <Directory /var/www/WebSecurityEnvironment2>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/lab5-2-error.log
    CustomLog ${APACHE_LOG_DIR}/lab5-2-access.log combined
</VirtualHost>
```

---

### `lab5-3.conf` — Hardened Site
```apache
<VirtualHost *:80>
    ServerAdmin mridul@lab5-3.webpentesting
    ServerName lab5-3.webpentesting
    DocumentRoot /var/www/WebSecurityEnvironment3

    <Directory /var/www/WebSecurityEnvironment3>
        Options -Indexes
        AllowOverride All
        Require all granted

        <FilesMatch "(^\.ht|\.git|\.env)">
            Require all denied
        </FilesMatch>

        <LimitExcept GET HEAD OPTIONS>
            Require all denied
        </LimitExcept>
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/lab5-3-error.log
    CustomLog ${APACHE_LOG_DIR}/lab5-3-access.log combined
</VirtualHost>
```

---

## 🔒 Security Hardening Explained

| Directive | What It Does | Why It Matters |
|---|---|---|
| `Options -Indexes` | Disables directory listing | Attackers cannot browse your file structure |
| `FilesMatch .ht/.git/.env` | Blocks sensitive files | Prevents leaking credentials and config |
| `LimitExcept GET HEAD OPTIONS` | Restricts HTTP methods | Blocks DELETE/PUT — prevents file writes & deletions |
| `ServerTokens Prod` | Hides Apache version | Stops automated scanners from targeting specific CVEs |
| `ServerSignature Off` | Removes version from error pages | Reduces information disclosure |

---

## 🛠️ Apache Management Commands

```bash
# Start / Stop / Restart / Reload
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2

# Check status
sudo systemctl status apache2

# Watch logs in real time
sudo tail -f /var/log/apache2/error.log
sudo tail -f /var/log/apache2/access.log

# Check journal for errors
sudo journalctl -u apache2 -f
```

---

## 🔐 SSL Certificate Generation

```bash
# Create directory
sudo mkdir /etc/ssl/localcerts

# Generate self-signed certificate (valid 365 days)
sudo openssl req -new -x509 -days 365 -nodes \
  -out /etc/ssl/localcerts/mridul.pem \
  -keyout /etc/ssl/localcerts/mridul.key

# Restrict access — root only
sudo chmod 700 /etc/ssl/localcerts/mridul.pem
sudo chmod 700 /etc/ssl/localcerts/mridul.key

# Enable SSL module
sudo a2enmod ssl
```

> ⚠️ Self-signed certificates are free but show a browser warning. For production, always use a trusted Certificate Authority like Let's Encrypt.

---

## 📋 Final Environment Summary

| URL | Protocol | Notes |
|---|---|---|
| `http://lab5.webpentesting` | HTTP → HTTPS | Permanently redirects to SSL |
| `https://lab5.webpentesting` | HTTPS | Main site with SSL certificate |
| `http://lab5-2.webpentesting` | HTTP | Different background (cover2.jpg) |
| `http://lab5-3.webpentesting` | HTTP | Hardened — restricted access & methods |

---

## 💡 Key Note on lab5.conf Design

I intentionally kept only the `Redirect` directive in `lab5.conf` because its sole purpose is to forward HTTP traffic to HTTPS. Having additional configuration like `DocumentRoot` alongside a `Redirect` causes a redirect loop — which I encountered and fixed during this lab. The actual site configuration is entirely managed by `lab5-ssl.conf` on port 443.

---

*Lab completed as part of Web Pentesting module — Howest Cybersecurity 2026* ©️Mridul Miha
