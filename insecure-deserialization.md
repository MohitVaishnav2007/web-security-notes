# Insecure Deserialization

---

## Category: Modifying serialized objects

**Approach**: take a look on session and try to reverse it using repeater's inspector -> found that with admin functionality, it write b:0 -> change the b:0 to b:1 (which shows true instead of false) -> 
move forward and get access to admin functionality.

**Why:**

**Tools:** Burpsuite (Repeater and inspector)
