# Access Control


---

## Category: Unprotected admin functionality with unpredictable URL

**Approach:** View page source → URL in JS.

**Why:** Unpredictable URL, still has to reach the admin's browser somehow → check what client side code was given, not just what's rendered.

**Tool:** Browser only

---

## Category: Unprotected admin functionality

**Approach:** Search for robots.txt

**Why:** Dev tried to hide path from search engines via robots.txt → but robots.txt itself is public and fixed-location → "not indexed" ≠ "not accessible", so always check robots.txt/sitemap.xml first.

**Tool:** browser only

---

## Category: User role controlled by request parameter

**Approach:** go for session cookie (unencrypted) and change admin value to true.

**Why:** Server trusted a client controlled cookie value for authorization instead of re-verifying role server-side → readability(plaintext) made it easy to find, but the flaw is "client says so" ≠ "server confirms it" → always ask: could a user just edit this value themselves?

**Note:** change or edit session cookic values in browser (not in burpsuite) changes your session going forward.

**Tool:** Burpsuite, Browser

---

**Note:** Access Control bugs = server trusts something the client sent instead of checking its own records. Always ask: what is this app assuming the user "wouldn't try"? Then try it.

---

## Category: User ID controlled by request parameter (IDOR).

**Approach:** search for url and change the id parameter.

**Why:** Dev assumed that login users would only ever request their own id → but the URL parameter alone decided whose data got returned, with no checks against who was actually logged in → lesson: servers must verifies requested data belong to the current session, never trust a client-supplied id.

**Tool:** Browser

---

## Category: User role can be modified in user profile

**Approach:** Log in as `wiener` -> navigate to My Account -> intercept the "update email" `POST /my-account/change-email` request in Burp Repeater -> add a new body parameter `roleid=2` alongside the existing email field -> send request -> reload account page -> `Admin panel` link now appears -> access `/admin` and delete `carlos`

**Why:** Server trusted client-supplied `roleid` on account update -> endpoint accepted an unexpected `roleid` field with no authorization check -> never assume a parameter is "hidden" just because the UI doesn't expose it; if the server accepts it, treat it as attacker-controlled.

**Tools:** Burp Suite (Repeater)

---
