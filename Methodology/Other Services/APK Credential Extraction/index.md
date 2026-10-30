# APK Credential Extraction

Decompile and grep — mobile apps frequently hardcode backend/domain credentials.

```bash
jadx -d jadx_out/ <app>.apk
grep -ir password jadx_out/
grep -ir -E 'api[_-]?key|secret|token' jadx_out/
```

