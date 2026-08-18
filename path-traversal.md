# Security Misconfiguration → File Path Traversal

## Path Traversal, a simple lab

**Approach:**
As known that `/image?filename` is vulnerable, so use that path to access passwd, by running:
```
/image?filename=../../../etc/passwd
```

**Why:** Filename parameter trusted as-is → `../` sequences walked the path outside the intended directory → always canonicalize and allowlist file paths built from user input.

**Tools:** Burpsuite, Browser

---
