# Security Misconfiguration

## CORS Vulnerability with basic origin reflection

**Approach:**
At proxy, > HTTP History, find the GET request to `/accountDetails` (this is AJAX call that fetches the API key). Send it to Repeater.

→ To test CORS misconfig → In Repeater, add or modify the header `Origin: https://evil-user.net` for testing. → Send it & check the response headers if you see `Access-Control-Allow-Origin` (reflecting back same origin), plus `Access-Control-Allow-Credentials: true`, it's confirmed the vulnerable.

→ Then, put a payload in body of "Go to exploit server".

→ Click "Store", then "Deliver to victim". Then, go to exploit server > "Access log". You should see a GET request to `/log?key=...` containing the victim's API key.

**Why:** Origin header treated as trusted → reflected verbatim into ACAO with credentials allowed → never let client-supplied input decide a trust boundary.

**Tools:** Burpsuite.

---


