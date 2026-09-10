# SSRF (Server-Side Request Forgery)

---

## Category: Basic SSRF against the local server

**Approach:** At proxy, > HTTP History, find the "Check stock" request — it sends a `stockApi` parameter that the server fetches server-side and returns the response from. Send it to Repeater.

→ Replace the `stockApi` value with `http://localhost/admin` — confirms the internal admin panel is reachable and returns its HTML.

→ Find the delete-user path on the panel (`/admin/delete?username=carlos`), and replace the `stockApi` value with `http://localhost/admin/delete?username=carlos`.

→ Send the request — the server fetches the internal URL on your behalf, executing the delete action. Solves the lab.

**Why:** No host validation on `stockApi` assumed to be safe since "we control the feature" → server blindly fetched whatever URL it was given, including `localhost` → any server-side "fetch this URL" feature needs an allowlist of destination hosts, not just a URL-format check.

**Tools:** Burpsuite (Repeater).

---

## Category: Basic SSRF against another back-end system

**Approach:** Same vulnerable `stockApi` parameter, but target IP is unknown this time. Send the "Check stock" request to Intruder.

→ Set the value to `http://192.168.0.§x§:8080/admin`, with `§x§` as the payload position.

→ Load a numbers payload list (0–255), run a Sniper attack.

→ Sort/compare responses — most return connection errors, one IP (`192.168.0.102`) returns a real response revealing a live admin panel on port 8080.

→ Repeat the Lab 1 technique: set `stockApi=http://192.168.0.102:8080/admin/delete?username=carlos` and send. Solves the lab.

**Why:** Same unvalidated `stockApi` host flaw as before, but developers assumed the internal network beyond localhost was a separate unreachable zone → Intruder swept the private IP range from the server's own vantage point, exposing a hidden internal host → SSRF isn't limited to the vulnerable server itself, it's a launchpad to scan and reach its entire reachable network — block all private IP ranges (RFC1918), not just `localhost`.

**Tools:** Burpsuite (Intruder)

---


