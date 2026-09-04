# XSS (Cross-Site Scripting)

## Category: Reflected XSS into HTML context (nothing encoded)

**Approach:** Noticed the `search` parameter in the URL (`?search=...`) gets reflected back into the page.

→ Replaced the search value with `<script>alert('get an alert')</script>` and submitted.

→ No encoding was applied — the script tag executed directly, popping the alert. Lab solved.

**Why:** The `search` parameter was assumed to only ever be displayed as plain text, so it was reflected into the HTML response without encoding → the `<script>` tag was parsed and executed by the browser instead of being treated as text → any user input rendered into HTML must be HTML-encoded before output.

**Tools:** Burpsuite (optional — browser alone is enough for this one).

---

## Category: Stored XSS into HTML context with nothing encoded

**Approach:** Identified the comment section as the vulnerable input — comments are stored server-side and rendered back to every visitor of the blog post.

→ Submitted a comment containing `<script>alert('congrats, the attack is successful')</script>` directly in the comment field.

→ Reloaded the page — the payload executed automatically for anyone viewing the post, confirming the injection was stored and unencoded. Lab solved.

**Why:** The comment field was assumed to only ever hold plain-text opinions, so it was stored raw and rendered into the page's HTML with no encoding → the script tag persisted in the database and executed on every page load for every visitor, not just the attacker → sanitize/encode all persisted user content before rendering, since stored XSS impacts every viewer.

**Tools:** Burpsuite (optional — browser alone is enough for this one)

---

## Category: DOM XSS in document.write sink using source location.search

**Approach:** Searched a term in the search box and inspected the rendered HTML — found the search term reflected inside an `<img>` tag's `src` attribute, written there client-side via the page's own `document.write()` call using `location.search` as the source.

→ Since the input landed inside an attribute value, closed the attribute and tag first, then injected a fresh script: `newpost"><script>alert('success on my foot')</script><`

→ Submitted this as the search query — the alert fired. Lab solved.

**Why:** Search term from location.search written raw into an img src via document.write → closing the attribute and tag broke out into a new <script> → sanitize/encode data client-side before it reaches any DOM sink, since the server never sees this class of payload.

**Tools:** Browser DevTools (Inspect Element) to trace the reflection point

---

## Category: DOM XSS in innerHTML sink using source location.search

**Approach:** Tested the search box and inspected where the search word gets reflected in the HTML — found it inside a span, inserted via `innerHTML`.

→ Tried `<script>alert('maybe it gee')</script>` first — it showed up in the DOM but didn't fire, since script tags are blocked from executing when inserted via `innerHTML` as a security measure.

→ Tried `<img src=x onerror=alert("error ditacted")>` — still didn't fire, since the double quotes in the payload were being mangled.

→ Removed the quotes and ran `<img src=x onerror=alert(1)>` — it worked. Lab solved.

**Why:** `innerHTML` blocks `<script>` tags from executing as a security measure, but it still renders event-handler attributes like `onerror` normally → the img's broken `src=x` triggered `onerror=alert(1)` → filtering `<script>` tags alone isn't enough, since HTML elements with event handlers still execute through `innerHTML`.

**Tools:** Browser DevTools (Inspect Element) to trace the reflection point

---
