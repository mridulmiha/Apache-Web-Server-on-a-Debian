## 🗺️ Step-by-Step Setup Guide

> **💡 Replace** `lab5`/`lab5.webpentesting` with your preferred name (e.g., `mysite`/`mysite.local`)**

---


 ### **📦 STEP 1 — Debian 13 VM Setup**
 ```bash
Downlord a debian linux and set up it on VMware/VmFusion/VisualBox
It would batter if you create a minimal linux without GUI just the CLI.
```
 ### **🔄 STEP 2 — System Update** 

```bash
sudo apt update && sudo apt upgrade -y
```

### **📦 STEP 3 — Install Production Stack**
```bash
sudo apt install apache2 php8.4 mariadb-server git tmux -y
```

### **🎰 STEP 4 — Verify Apache + PHP**
```bash
# Test Apache default page
In browser on localhost http://(vm ip)/ | Output "It works!"

# PHP verification
sudo nano /var/www/html/info.php  | '<?php echo phpinfo (); ?>'  | search info.php -
http:// VM ip/info.php | see the "PHP Version"
  # php 8.4✅
```

### **🌐 STEP 5 — Clone Vulnerable Web App**
```bash
cd /var/www/
sudo git clone https://github.com©️KoenKoreman (You can use your Preferred repo)
ls -la WebSecurityEnvironment/  
# ✅ Folder created/added
```

### **🖥️ STEP 6 — Virtual Host 1 (lab5.conf)**
```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/lab5.conf
sudo nano /etc/apache2/sites-available/lab5.conf
```
**Edit `lab5.conf`:**
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@lab5.webpentesting
    ServerName lab5.webpentesting
    DocumentRoot /var/www/WebSecurityEnvironment
    
    <Directory /var/www/WebSecurityEnvironment>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
```bash
sudo a2ensite lab5.conf && sudo systemctl restart apache2
```

---

### **🌐 STEP 7 — Client Hosts File**
**Windows (PowerShell as Admin):**
```bash
notepad "C:\Windows\System32\drivers\etc\hosts"
```
**Add line:** `YOUR_VM_IP lab5.webpentesting`

**Linux/Mac:**
```bash
echo "YOUR_VM_IP lab5.webpentesting" | sudo tee -a /etc/hosts
```

**Test:** `curl -I http://lab5.webpentesting/`

---

### **🖥️ STEP 8 — Virtual Host 2 (Different Background)**
```bash
sudo cp -r /var/www/WebSecurityEnvironment /var/www/WebSecurityEnvironment2

sudo sed -i 's/cover.jpg/cover2.jpg/g' /var/www/WebSecurityEnvironment2/assets/css/style.css
(Or you can do manually with nano)

sudo cp /etc/apache2/sites-available/lab5.conf /etc/apache2/sites-available/lab5-2.conf

sudo nano /etc/apache2/sites-available/lab5-2.conf  # ServerName: lab5-2.webpentesting

sudo a2ensite lab5-2.conf && sudo systemctl restart apache2
```
**Add to hosts:** `YOUR_VM_IP lab5-2.webpentesting`

---

### **🔒 STEP 9 — Virtual Host 3 (HARDENED Security)**
```bash
sudo cp -r /var/www/WebSecurityEnvironment /var/www/WebSecurityEnvironment3

sudo cp /etc/apache2/sites-available/lab5.conf /etc/apache2/sites-available/lab5-3.conf

sudo nano /etc/apache2/sites-available/lab5-3.conf
```
**Add Security Directives:**
```apache
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
```
```bash
sudo a2ensite lab5-3.conf && sudo systemctl restart apache2
```
**Add to hosts:** `YOUR_VM_IP lab5-3.webpentesting`

---

### **🔐 STEP 10 — SSL Certificate**
```bash
sudo mkdir -p /etc/ssl/localcerts

cd /etc/ssl/localcerts

sudo openssl req -new -x509 -days 365 -nodes \
  -out server.pem -keyout server.key
  
  (You can replace the name SERVER with your choice but best practice is go with the domain name)

sudo chmod 700 server.*
```

### **🔐 STEP 11 — HTTPS Virtual Host**
```bash
sudo a2enmod ssl

sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/lab5-ssl.conf

sudo nano /etc/apache2/sites-available/lab5-ssl.conf
```
**SSL Paths:**
```apache
SSLCertificateFile /etc/ssl/localcerts/server.pem
SSLCertificateKeyFile /etc/ssl/localcerts/server.key
```
```bash
sudo a2ensite lab5-ssl.conf && sudo systemctl restart apache2
```

---

### **🔄 STEP 12 — HTTP → HTTPS Redirect (CRITICAL!)**
```bash
sudo nano /etc/apache2/sites-available/lab5.conf
```
**KEEP ONLY:**
```apache
<VirtualHost *:80>
    ServerName lab5.webpentesting
    Redirect permanent / https://lab5.webpentesting/
</VirtualHost>
```
```bash
sudo systemctl restart apache2
```

### **✅ STEP 13 — Production Tests**
```bash
curl -I http://lab5.webpentesting/      # → 301 HTTPS ✅
curl -I https://lab5.webpentesting/    # → 200 SSL ✅
curl -I http://lab5-2.webpentesting/   # → 200 Background ✅
curl -I http://lab5-3.webpentesting/   # → 200 Hardened ✅
```
