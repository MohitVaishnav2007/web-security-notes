# Insecure Deserialization

---

## Category: Modifying serialized objects

**Approach**: take a look on session and try to reverse it using repeater's inspector -> found that with admin functionality, it write b:0 -> change the b:0 to b:1 (which shows true instead of false) -> 
move forward and get access to admin functionality.

**Why:** Assumed base64+serialization obscurity = security -> admin flag sat in plaintext-decodable serialized cookie with no signing -> never trust client-held serialized state for auth decisions; sign/encrypt it or re-derive it server-side

**Tools:** Burpsuite (Repeater and inspector)
