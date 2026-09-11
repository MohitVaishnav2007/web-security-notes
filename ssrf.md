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

## Category: SSRF with Blacklist-based input filter

**Approach:** lab runs two separate blacklist checks on the stockApi value:
-> One matching the host string — blocks literal 127.0.0.1, localhost, etc.
-> One matching the path string — blocks the literal word admin.

as know that two filter are applied, for both address and path, we use alternate for both -> admin instead of admin and 127.1 instead of 127.0.0.1. 
So applying at stockApi http://127.1/Admin/delete?username=carlos , invaliding both the host string and path string blacklist checks. 

**Why:** Blacklist matches raw strings 127.0.0.1/admin → attacker sends semantically-identical-but-textually-different 127.1/Admin → server's parser/router resolves both anyway, bypassing both filters at once → fix: validate against the resolved IP/route post-parsing, not pre-parsing string comparison, or better, switch to an allowlist entirely.

**Tools:** Burpsuite (repeater)

---

## Category: SSRF with Whitelist-based input filter

**Approach:** Step 1 — http://127.0.0.1/
Rejected with a specific message: "External stock check host must be stock.weliketoshop.net". This told us the filter isn't a dumb substring search (unlike Lab 3) — it's actually parsing the URL structure and extracting a real host field, then comparing it. A more sophisticated defense than the blacklist labs.

Step 2 — http://username@stock.weliketoshop.net/
Accepted, but then crashed with a 500. This was the first crack: the filter correctly read username as userinfo and stock.weliketoshop.net as the host — so it passed validation. But the fetcher, making the real connection, apparently read the URL differently and tried to connect to username itself (which doesn't exist as a domain) — hence the crash. This proved the two parsers can disagree, even if we hadn't yet found a way to make that disagreement useful.

Step 3 — http://username#@stock.weliketoshop.net/
Rejected. Here we introduced #, which in proper URL syntax starts the fragment — everything after it is meant to be ignored entirely by both the host-resolution step and the server (fragments never even get sent in the actual HTTP request; they're a browser-local concept). If the fetcher treated # as a real fragment delimiter, the effective host would become username, and @stock.weliketoshop.net/ would just be a discarded fragment. The filter rejected this — meaning at this point, the filter is reading # correctly as a fragment marker too, and correctly determining the real host is username — not the whitelisted domain — so it blocked it. Both components agreed here.

Step 4 — http://username%23@stock.weliketoshop.net/
Still rejected, same message. %23 is the URL-encoded form of #. This tells us the filter decodes percent-encoding once before doing its host check — so %23 and a literal # look identical to it. Still no disagreement yet; both components (as far as we can tell) are on the same page.

Step 5 — http://username%2523@stock.weliketoshop.net/
500 error. %2523 is # double-encoded (%25 = encoded %, so %2523 = encoded %23 = doubly-encoded #). This is where the disagreement finally appeared:

The filter decoded it once: %2523 → %23. That's not a literal # yet, just a harmless-looking string — so the filter's host-check saw username%23 as part of the userinfo and stock.weliketoshop.net as the host. Passed.
The fetcher, when actually opening the connection, decoded it a second time: %23 → #. Now, at this layer, the # really does act as a fragment delimiter — so the fetcher's real host became username, and everything after # was dropped. It tried (and failed) to connect to username — hence the 500.

That 500 was proof the fetcher decodes one extra layer deeper than the filter does — a genuine parser differential, purely from how many times each component runs percent-decoding.

Step 6 (final) — http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
Same trick, but now with the real destination in place of the placeholder username. The filter still sees localhost:80%23... as userinfo and stock.weliketoshop.net as host → passes. The fetcher double-decodes %2523 → #, treats localhost:80 as the real host, connects there, and reaches the internal admin panel — executing the delete.

**Why:** Filter decodes %2523 once (still looks safe) → fetcher decodes it twice, revealing a real # that turns everything after it into a discarded fragment → real connection goes to localhost:80 instead of the whitelisted host → fix: validate the URL after full/final decoding, using the exact same parsing logic the fetcher will use — ideally the same function/library, not two independent implementations.

**Tools:** Burpsuite

---
