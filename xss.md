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
