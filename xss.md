# XSS (Cross-Site Scripting)

## Category: Reflected XSS into HTML context (nothing encoded)

**Approach:** Noticed the `search` parameter in the URL (`?search=...`) gets reflected back into the page.

→ Replaced the search value with `<script>alert('get an alert')</script>` and submitted.

→ No encoding was applied — the script tag executed directly, popping the alert. Lab solved.

**Why:** The `search` parameter was assumed to only ever be displayed as plain text, so it was reflected into the HTML response without encoding → the `<script>` tag was parsed and executed by the browser instead of being treated as text → any user input rendered into HTML must be HTML-encoded before output.

**Tools:** Burpsuite (optional — browser alone is enough for this one).
