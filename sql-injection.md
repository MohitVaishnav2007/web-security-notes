# SQL Injection

---

## Category : Vulnerability in WHERE Clause Allowing Retrieval of Hidden Data

**Approach:** Using `GIFTS'+OR+1=1--` in the category filter to bypass the following query and comment out the rest — letting all hidden/unlisted data of the query return.

**Why:** Dev assumed that client input was just data → string concatenation let it be parsed as SQL syntax instead, breaking out of the WHERE clause and overriding the query's logic → lesson: always use parameterized queries so input can never be interpreted as code.

**Tool:** Browser (URL)

---

## Category : Vulnerability Allowing Login Bypass

**Approach:** Using `Administrator'--` to login as administrator and bypassing the password check by commenting remaining query.

**Why:** Dev assumed the password check would always run alongside the username check → string concatenation let the injected comment strip the password condition out of the query entirely → lesson: authentication checks built via string concatenation can be structurally removed by attacker input, so always use parameterized queries to keep query logic fixed regardless of input.

**Tool:** Browser (URL)

---

## Category : UNION Attack, Determining the Number of Columns Returned by the Query

**Approach:** In the category filter, incrementally test `'+ORDER+BY+n--` (n=1,2,3...) until the server throws an error. Last successful number is the column count. Confirm it by running `'+UNION+SELECT+NULL,NULL,NULL--` (NULL = matching the count) — returns without error. Then swap individual NULLs for a string like `'a'` to find which columns are actually reflected on the page.

**Why:** Dev assumed raw DB errors were harmless noise → but the app let SQL syntax errors reach the response, so success vs failure on incrementing ORDER BY/UNION column counts leaked the exact number of columns in the query → lesson: never expose raw database errors, since even a binary success/fail signal is enough to map internal query structure.

**Tool:** Browser (URL)

---

## Category : UNION Attack, Finding a Column Containing Text

**Approach:** Determining the number of columns returned by the query in category filter → replacing NULL by string like `'a'` to determine the data type of column: `'+UNION+SELECT+NULL,'a',NULL--`

**Why:** Dev assumed a working UNION query didn't mean controllable output → but swapping NULLs for a unique string marker showed which column's data actually rendered on the page → lesson: a single type-compatible, reflected column is enough to turn UNION injection into a working data-exfiltration channel.

**Tool:** Browser (URL)

---

## Category : UNION Attack, Retrieving Data from Other Tables

**Approach:** As we know product category filter is vulnerable, and there's a table called users, having two fields/columns: username and password, so: `'+UNION+SELECT+username,+password+FROM+users--`

**Why:** Dev assumed UNION could only touch the current table → but matching column count/type let the injected SELECT pull from `users` instead → lesson: once columns are known, UNION lets an attacker read any table the DB user can access.

**Tool:** Browser (URL) or Burp Suite

---
