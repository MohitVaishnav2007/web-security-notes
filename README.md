# web-security-notes

Personal notes and cheat sheets from web security labs (PortSwigger Web Security Academy, TryHackMe, etc.), organized by vulnerability category.

Each file covers one category. Every solved lab gets logged as: **Approach → Why → Tool**.

## Categories

- [Access Control](./access-control.md)
- [SQL Injection](./sql-injection.md)
- [Security Misconfiguration](./security-misconfiguration.md)
- [Path Transversal](./path-traversal.md)
- [Authentication Failures](./authentication-failures.md)
- [Cryptographic Failures](./cryptographic-failures.md)
- [XSS(Cross-Site Scripting)](./xss.md)
- [SSRF(Server-Side Request Forgery)](./ssrf.md)
- [CSRF(Cross-Side Request Forgery)](./csrf.md)
- [Insecure Deserialization](./insecure-deserialization.md)

## Format

Each entry follows this structure:

```markdown
## Category: <Category> → <Lab name>

**Approach:** <what you did>

**Why:** <why it worked / the underlying concept>

**Tool:** <tools used>
```
