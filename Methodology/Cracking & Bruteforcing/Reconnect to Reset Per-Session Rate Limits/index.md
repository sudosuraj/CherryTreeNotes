# Reconnect to Reset Per-Session Rate Limits

Some rate limits are tied to the connection/session, not the account or IP. If a service caps attempts per socket/session (e.g. "3 tries then disconnect"), open a fresh connection for every single guess instead of trying to defeat the limit directly — it resets the counter for free.
