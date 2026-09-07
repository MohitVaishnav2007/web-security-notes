# Cryptographic Failures

---

## Category: JWT Authentication Bypass via Unverified Signatures

**Approach**: Taking login with wiener:peter (given) -> search for appropriate http request history containing the JWT token and send it to repeater -> edit the JWT token using the JWT editor in Burpsuite's repeater and after sign that token and checks it's working -> replace in all the places at the original token (at the time of regaining the login).

**Why**: Server trusted JWT claims without verifying the signature → editing the payload to claim "administrator" was accepted as-is → always cryptographically verify a JWT's signature server-side before trusting any of its claims.

**Tools**: Burpsuite (JWT Editor extension) 

---
