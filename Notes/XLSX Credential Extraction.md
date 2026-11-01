# XLSX Credential Extraction

`.xlsx` files are ZIP archives. Strings (including credentials) are stored in `xl/sharedStrings.xml`.

```bash
# Extract and dump all shared strings
unzip accounts.xlsx -d xlsx_extracted
xmllint --xpath '//*[local-name()="t"]/text()' xlsx_extracted/xl/sharedStrings.xml

# Format as CSV rows (adjust the modulus to match column count, e.g. 5 for 5 columns)
xmllint --xpath '//*[local-name()="t"]/text()' xlsx_extracted/xl/sharedStrings.xml \
  | awk 'ORS=NR%5?",":"\n"'
```

If `xmllint` is unavailable:
```bash
python3 -c "
import zipfile, re
with zipfile.ZipFile('accounts.xlsx') as z:
    data = z.read('xl/sharedStrings.xml').decode()
    print('\n'.join(re.findall(r'<t[^>]*>([^<]+)</t>', data)))
"
```

Check other XML files too — `xl/worksheets/sheet1.xml` contains cell values not stored as shared strings (numbers, dates, inline strings with `t=\"inlineStr\"`).
