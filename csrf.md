# CSRF (Cross-Site Request Forgery)

---

## Category: CSRF vulnerability with no defenses

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

## Category: CSRF where token validation depends on request method

**Approach:** Same `/my-account/change-email` endpoint, but now a CSRF token parameter is required and validated — on `POST` requests only.

→ Confirm a normal `POST` with a missing/invalid `csrf` value gets rejected.

→ Confirm the endpoint still accepts `GET` requests with the same parameters passed as a query string, and performs the same action.

→ Build the PoC form with `method="GET"` instead of `POST`, still including the (irrelevant) `csrf` hidden input:
```html
<form id="csrf-form" action="https://<lab-id>.web-security-academy.net/my-account/change-email" method="GET">
    <input type="hidden" name="email" value="attacker@example.com">
    <input type="hidden" name="csrf" value="anything">
</form>
<script>
    document.getElementById("csrf-form").submit();
</script>
```

→ Store on exploit server, view/deliver — GET request bypasses the token check entirely (validation only runs in the POST code path) and the email changes. Solves the lab.

**Why:** CSRF token validated only on the `POST` handler → same endpoint still accepts `GET` and performs the same action → attacker's form uses `method="GET"` instead, skipping the validation path entirely → fix: validate the CSRF token for every method/route that can trigger the state-changing action, or explicitly reject any method other than the one intended.

**Tools:** Burpsuite (Repeater), PortSwigger exploit server, hand-written HTML/JS.

---

## Category:  CSRF where token validation depends on token being present

**Approach**: As from the lab, we already know that the change email functionality is CSRF vulnerable so we take a test by removing the entire input field from our payload and it gets succeeded, demonstrating that the lab only validates the token if it present. 

→ Build the PoC form with `method="POST"` by removing the csrf input field.
```html
<form id="csrf-form" action="https://<lab-id>.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="attacker@example.com">
</form>
<script>
    document.getElementById("csrf-form").submit();
</script>
```

**Why**: CSRF token only validated if the parameter exists in the request → omit the csrf field entirely from the forged form → server's "if present, validate" logic has no else-branch rejecting absence → fix: require the token parameter unconditionally — treat a missing token exactly the same as an invalid one (both = reject)

**Tools**: Burpsuite(Proxy/Repeater), PortSwigger exploit server, hand-written HTML/JS


---


## Category: CSRF where token is not tied to user session

**Approach:** Same `/my-account/change-email` endpoint, CSRF token now required and validated — but only checked as "is this token valid and unused," never "does this token belong to the current session."

→ Log in as `wiener:peter`, go to account page, grab the CSRF token from the hidden field in the change-email form (view page source or intercept `GET /my-account` response).

→ Build the PoC form with `method="POST"`, wiener's freshly grabbed token hardcoded into the `csrf` hidden field:
```html
<form id="csrf-form" action="https://<lab-id>.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="attacker@example.com">
    <input type="hidden" name="csrf" value="WIENERS_FRESH_TOKEN_HERE">
</form>
<script>
    document.getElementById("csrf-form").submit();
</script>
```

→ Store on exploit server. **Do NOT click "View exploit"** — that would burn the token before delivery. Go straight to **"Deliver exploit to victim."**

→ Carlos's browser loads the page, auto-submits with his session cookie + wiener's valid token → server validates both independently, accepts the request, changes carlos's email. Solves the lab.

**Why:** Token validated as "exists and unused" but never checked against the requesting session → attacker extracts their own valid token and hardcodes it into the forged form → victim's browser submits attacker's token + victim's cookie → server accepts both independently → fix: bind every token to the session ID at issuance time and reject any token that doesn't match the current session on validation.

**Tools:** Burpsuite (Proxy/Repeater), PortSwigger exploit server, hand-written HTML/JS

---
