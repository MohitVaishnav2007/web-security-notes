# Business Logic Vulnerability/Flaws


---

## Category: Excessive trust in client-side controls

**Approach:** Log in as `wiener` -> add item to cart -> intercept "add to cart" request in Burp Proxy -> modify the price parameter to an arbitrary lower value -> forward request -> complete checkout at tampered price

**Why:** Server assumed the client-submitted price could be trusted since it originated from its own front-end form -> no server-side re-validation against the actual stored product price occurred at checkout -> never trust any value from the client, even one your own UI generated; always re-derive/verify price-affecting data server-side.

**Impact:** Allows any authenticated user to purchase items at an arbitrary attacker-chosen price, resulting in direct financial loss to the business.

**Tools:** Burp Suite (Proxy)

---

## Category: 2FA broken logic

**Approach:** Log in to your own account with Burp running and investigate the 2FA flow. Notice the `POST /login2` request uses a `verify` parameter to determine which user's account the 2FA code applies to.

→ Log out. Send the `GET /login2` request to Repeater, change `verify` to `carlos`, and send it — this generates a temporary 2FA code for Carlos.

→ Go to the login page, log in with your own username/password, then submit any invalid 2FA code to capture the `POST /login2` request.

→ Send that `POST /login2` request to Intruder. Set `verify=carlos` and add a payload position on the `mfa-code` parameter.

→ Use a **Numbers** payload (From: `0`, To: `9999`, Min integer digits: `4`) to brute-force all possible 4-digit codes. Run Sniper attack.

→ Sort results by status code — the request returning `302` has the correct code. Load that response in the browser and click **My account** to solve the lab.

**Why:** The app trusted the `verify` parameter to scope which account a 2FA code belonged to, but never checked whether the *same user* who requested the code was the one submitting it → an attacker could generate a valid code for another account and brute-force it without rate limiting → 2FA must be bound to the authenticated session that requested it, and code submission must be rate-limited.

**Tools:** Burpsuite.

---

## Category: High-level logic vulnerability

**Approach:** Log in as `wiener` -> add "Lightweight l33t leather jacket" to cart at normal quantity (1) -> add a separate cheap item to cart -> intercept the request updating that cheap item's quantity -> set it to a large negative value -> forward request -> cart total drops below available store credit -> place order

**Why:** Cart total was computed by summing per-item price × client-supplied quantity with no bound on negative values -> a negative quantity on a cheap item produced a negative line total, dragging the overall cart total below available credit -> server must validate that quantities stay within sane bounds (e.g. ≥0) before trusting any arithmetic built from them.

**Impact:** Allows any authenticated user to acquire high-value items essentially for free by manipulating cart quantities, resulting in direct financial loss.

**Tools:** Burp Suite (Proxy/Repeater)

---
