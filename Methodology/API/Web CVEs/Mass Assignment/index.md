# Mass Assignment

If a framework binds the whole request body straight onto a model (e.g. Spring `@ModelAttribute`) with no field allow-list, add fields it shouldn't let you set.

```bash
# intercept a legit "update profile" request in Burp, add an extra field
role=<ADMIN_ROLE_VALUE>
```

