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


