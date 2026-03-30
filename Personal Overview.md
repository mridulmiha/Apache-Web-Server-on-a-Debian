## 🎯 What I Learned

| Topic | Key Takeaway |
|---|---|
| Virtual Hosts | `ServerName` must match the client hosts file exactly |
| HTTP → HTTPS | Only put `Redirect` in the HTTP config — nothing else |
| SSL Certificate | `.pem` = public key, `.key` = private key, always `chmod 700` both |
| Security Hardening | Always disable directory listing on any production server |
| Live Logs | `tail -f` is essential for real-time debugging |
| tmux | Split panes with `Ctrl+B "` and navigate with `Ctrl+B ↓` |

---

## ⚡ Challenges I Faced

### 🔴 Challenge 1 — Redirect Loop (`ERR_TOO_MANY_REDIRECTS`)
**Problem:** After setting up HTTPS, the site kept looping and gave `ERR_TOO_MANY_REDIRECTS`.  
**Root Cause:** I had `DocumentRoot` and `Redirect` in the same HTTP config, which caused Apache to loop.  
**Fix:** Removed everything from `lab5.conf` except the `Redirect` directive. The actual site is entirely handled by `lab5-ssl.conf` on port 443.

```apache
# ❌ Wrong — causes redirect loop
<VirtualHost *:80>
    DocumentRoot /var/www/WebSecurityEnvironment
    Redirect permanent / https://lab5.webpentesting/
</VirtualHost>

# ✅ Correct — clean redirect only
<VirtualHost *:80>
    ServerName lab5.webpentesting
    Redirect permanent / https://lab5.webpentesting/
</VirtualHost>
```

---

### 🔴 Challenge 2 — Windows Hosts File Permission Denied
**Problem:** Could not save the Windows hosts file — `You don't have permission to save in this location`.  
**Fix:** Opened PowerShell as Administrator and ran:
```powershell
notepad C:\Windows\System32\drivers\etc\hosts
```
This opens Notepad with admin rights and allows saving directly to the hosts file.

---

### 🔴 Challenge 3 — Virtual Host Showing Wrong Content
**Problem:** `cat lab5.conf` was showing `default-ssl.conf` content instead of my config.  
**Root Cause:** The file was copied from the wrong source and never properly overwritten.  
**Fix:** Used `nano` to clear the entire file with `Ctrl+K` and rewrote it from scratch.

---
