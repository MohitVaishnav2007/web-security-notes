# Authentication Failures


---

## Category: Username enumeration via different responses

 **Approach**: At proxy, > HTTP History, find the `POST` request to `/login`. Send it to Intruder.

→ Load the username wordlist into the `username` parameter, keep `password` fixed to any random value. Run Sniper attack.

→ Sort/compare responses in the results grid — one username shows a different response length, and its response body reads `"Incorrect password"` instead of `"Invalid username"`. Note this username.

→ Switch attack: fix the `username` field to the noted value, load the password wordlist into the `password` parameter. Run Sniper attack.

→ Filter results by status code — the request returning `302` is the correct password. Log in with that pair to solve the lab.

**Why:** Same-looking error message assumed to hide account existence → response length and body text still differed between "user doesn't exist" and "user exists, wrong password" → never let *any* observable difference (text, length, timing, status) between the two cases reach the client.

**Tools:** Burpsuite

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

## Category: Password reset broken logic

**Approach:** Test the password reset flow and found that the `temp-forgot-password-token` parameter is not actually being validated by the server (verified by removing its value entirely).

→ Request a new password reset with the token removed at `POST /forgot-password?temp-forgot-password-token=` — both the URL parameter and the request body's token field left empty.

→ Change the `username` field to `carlos` and set the `password` field to whatever you want.

→ Send the request — the server accepts it and resets Carlos's password without ever checking the token's validity.

**Why:** The presence of a `temp-forgot-password-token` parameter was assumed to mean the token had been validated → the server never actually checked whether the token was correct (or even present) before honoring the reset, only reading the `username` field → any sensitive action gated by a token must explicitly verify that token server-side, an empty or missing value should never be silently accepted.

**Tools:** Burpsuite.

---
