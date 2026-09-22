# Buusiness Logic Vulnerability/Flaws


---

## Category: Excessive trust in client-side controls

**Approach:** Log in as `wiener` -> add item to cart -> intercept "add to cart" request in Burp Proxy -> modify the price parameter to an arbitrary lower value -> forward request -> complete checkout at tampered price

**Why:** Server assumed the client-submitted price could be trusted since it originated from its own front-end form -> no server-side re-validation against the actual stored product price occurred at checkout -> never trust any value from the client, even one your own UI generated; always re-derive/verify price-affecting data server-side.

**Impact:** Allows any authenticated user to purchase items at an arbitrary attacker-chosen price, resulting in direct financial loss to the business.

**Tools:** Burp Suite (Proxy)

---
