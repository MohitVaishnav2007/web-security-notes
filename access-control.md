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

## Category: URL based access control can be circumvented

**Approach:** Log in as `wiener` -> request `/admin` directly -> blocked -> in Repeater, change the request path to `/` and add header `X-Original-URL: /admin` -> send -> admin panel now loads -> find the delete-user function's path (`/admin/delete`) -> send request to `/` with `X-Original-URL: /admin/delete` and query string `?username=carlos` -> `carlos` deleted

**Why:** Access control was enforced only on the visible request path by a front-end/proxy layer -> back-end also honored the `X-Original-URL` override header with no re-check, and separately read the query string as normal request arguments -> path-based access control must be enforced at the layer that actually serves the request, not a layer in front of it that can be bypassed via alternate routing headers; the override header only decides which handler runs, the query string still supplies that handler's arguments independently.

**Tools:** Burp Suite (Repeater)

---

## Category: Method-based access control can be circumvented

**Approach:** Log in as `wiener` -> capture the privileged `PUT /admin-roles` (username/action in body) request -> on wiener's own session, confirm blocked with `401 Unauthorized` -> change the HTTP method to an invalid, made-up verb (`LOLO`), keeping params in body -> still blocked/"missing parameter" -> convert request to GET format, moving `username=wiener&action=upgrade` into the query string while keeping the invalid method `LOLO`, still on wiener's own session -> send -> `302 Found` redirect to `/admin`, role upgraded

**Why:** Access control matched only a known whitelist of real HTTP methods -> route handler executed the action for any method string as long as it could parse the parameters -> access control must default-deny for anything that isn't explicitly allowed, not just check for specific known methods.

**Tools:** Burp Suite (Repeater)

---

## Category: User ID controlled by request parameter, with unpredictable user IDs

**Approach:** Find a post authored by `carlos` on the blog -> view page source of that post -> locate carlos's unique (GUID-style) user ID embedded in the HTML -> submit that user ID in place of your own in the relevant Burp request -> response returns carlos's API key -> submit the API key to solve the lab

**Why:** Unpredictable ID treated as auth -> ID leaked in page HTML -> obscurity isn't access control.

**Tools:** Burp Suite

---

## Category: User ID controlled by request parameter with data leakage in redirect

**Approach:** Log in as `wiener` -> capture the GET request that loads account/user data (`id=wiener` param) -> change `id` parameter to `carlos` -> server issues a redirect (302) instead of directly rendering the page -> inspect the redirect response closely -> carlos's API key is leaked in the redirect response body/headers before the redirect completes -> submit API key to solve lab

**Why:** Unpredictable/relevant authorization checked on final page render -> API key leaked in redirect response -> secure the data at every point it's transmitted, not just the final destination.

**Tools:** Burp Suite

---

## Category: User ID controlled by request parameter with password disclosure

**Approach:** Log in as `wiener` -> access "update account" functionality -> tamper the `id`/user parameter to point at `administrator` -> submit the update request without changing the password field -> intercept in Burp -> administrator's current password is disclosed in the request/response -> log in as `administrator` using that password -> delete `carlos` to solve the lab

**Why:** Access check only verified you were logged in as *some* user, not that the ID in the request matched your own -> the edit-account form pre-filled and sent back the target's existing password even when unchanged -> never trust "logged in" as equal to "authorized for this specific record," and never let a form leak an existing sensitive value just because it's rendering it for editing.

**Tools:** Burp Suite

---

## Category: Insecure direct object references

**Approach:** Open live chat -> download own transcript, note filename pattern (sequential integer + `.txt`) -> manually request an earlier file (`1.txt`) by editing the URL -> retrieve carlos's chat transcript, containing his password in plaintext -> log in as carlos with the leaked password

**Why:** Transcript filenames used a predictable sequential counter with no ownership check -> decrementing the number returned another user's file directly -> sequential/incremental IDs are guessable; always pair them with a server-side ownership check.

**Tools:** Browser, manual URL editing

---

## Category: Multi-step process with no access control on one step

**Approach:** Log in as `administrator` -> begin the "upgrade user" flow, capturing both requests (step 1: initiate/confirm, step 2: apply the action) in Burp -> log in as `wiener` in a separate session -> take step 2's request from the admin's captured flow, swap the session cookie to wiener's -> send step 2 directly, skipping step 1 -> wiener is upgraded to administrator

**Why:** Access control was only enforced on step 1 -> step 2's state-changing request had no independent check, trusting step 1 had already gated it -> every step of a multi-step flow must independently verify authorization, not just the entry point.

**Tools:** Burp Suite (Repeater)

---

## Category: Referer-based access control

**Approach:** Log in as `administrator` -> browse to admin panel, promote a user, capture that upgrade request in Burp (carries `Referer: .../admin` since it genuinely originated there) -> log in as `wiener`, swap the session cookie into the captured request, change `username` to `wiener` -> forward request (Referer header from admin's flow still attached) -> wiener promoted to administrator

**Why:** Access control was based on the `Referer` header instead of session/role -> the app trusted "request came from a page inside /admin" as proof of being an admin, but that header is fully client-controlled -> a request's origin (as claimed by a client-sent header) is never proof of authorization; enforce checks on server-verified identity, not headers the client can freely spoof.

**Tools:** Burp Suite (Repeater)

---
