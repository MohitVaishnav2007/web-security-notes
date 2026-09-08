# Authentication Failures


---
 
## Category: Brute-forcing a stay-logged-in cookie

**Approach:** Logged in as `wiener:peter` with "Stay logged in" checked, and inspected the `stay-logged-in` cookie in Burp.

→ Decoded it as Base64 in Burp Decoder — got `wiener:51dc30ddc473d43a6011e9ebba6ca770`, a username followed by a 32-character hex string.

→ Confirmed the hex string was MD5 by hashing the known password `peter` and matching it against the decoded value.

→ Logged out to remove the active `session` cookie, then sent a `GET /my-account?id=carlos` request to Intruder with the `stay-logged-in` value as the payload position.

→ Set the candidate password list as payloads, and added three Payload Processing rules in order: **Hash (MD5) → Add prefix (`carlos:`) → Encode (Base64)** — replicating the cookie's exact construction.

→ Ran the attack; the response that differed from the rest (correct redirect/length) revealed the working forged cookie. Used it to access Carlos's account and solve the lab.

**Why:** The `stay-logged-in` cookie was just `base64(username:md5(password))` → structure was reverse-engineered and rebuilt offline for any username with a guessable password → persistent auth tokens must use unpredictable, random values, never a reversible encoding of guessable credentials.

**Tools:** Burpsuite (Decoder, Intruder).

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

## Category: Basic password reset poisoning

**Approach:** Request a password reset for your own account (`wiener`) and send the `POST /forgot-password` request to Repeater.

→ Change the `Host` header to your exploit server's domain and set `username=carlos`. Send the request.

→ Check your exploit server's Access Log for a `GET /forgot-password?temp-forgot-password-token=...` request — this is Carlos's reset token.

→ Take a genuine reset link from your own email and swap in Carlos's token. Visit it, set a new password, and log in as `carlos`.

**Why:** The `Host` header was trusted to build the password reset link → an attacker-controlled Host redirected the victim's reset token to the attacker's server → always hardcode the domain server-side for sensitive links, never derive it from request headers.

**Tools:** Burpsuite.

---

## Category: Password reset poisoning via middleware

**Approach:** Trigger a password reset for `wiener` and send `POST /forgot-password` to Repeater. Plain `Host` header tampering doesn't change the reset link this time — the app sits behind middleware that ignores it.

→ Add a new header `X-Forwarded-Host: <your-exploit-server-id>.exploit-server.net` to the request, keeping the real `Host` header intact.

→ Change `username` to `carlos`. Send the request.

→ Check your exploit server's Access Log for `GET /forgot-password?temp-forgot-password-token=...` — Carlos's token.

→ Swap that token into your own genuine reset link, visit it, set a new password, and log in as `carlos`.

**Why:** Devs hardened against direct `Host` spoofing, but the middleware/reverse proxy still trusted `X-Forwarded-Host` to build the reset link → an attacker-controlled value in that header redirected Carlos's token to the attacker's server → all proxy-forwarding headers (not just `Host`) must be validated or stripped before use in sensitive URLs.

**Tools:** Burpsuite.


---
