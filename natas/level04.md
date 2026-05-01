# Natas Level 4 → 5

## Goal

The page rejects you: *"Access disallowed. You are visiting from 'X' while authorized users should come only from 'http://natas5.natas.labs.overthewire.org/'."* The server is trusting the `Referer` HTTP header to decide who's allowed. You control your own headers, so set it to what it wants.

## Concepts you'll learn

- The **Referer** header: your browser usually sends it automatically, and servers *should not* trust it for security
- How to override a header with `curl -H` or with Burp Suite / browser dev tools
- Why header-based allowlists are effectively worthless against any adversary

## Walkthrough

### 1. See the check fail

Open `http://natas4.natas.labs.overthewire.org/` as `natas4` with the level-3 password. The page shows the "access disallowed" message along with the expected `Referer`: `http://natas5.natas.labs.overthewire.org/`.

### 2. Send the request with the expected Referer

With `curl`:

```bash
$ curl -u natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ \
       -H "Referer: http://natas5.natas.labs.overthewire.org/" \
       http://natas4.natas.labs.overthewire.org/
```

Look for the password in the output:

```
The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq
```

### 3. (Alternative) Browser-based

If you prefer a GUI:

- Open the page, then open dev tools (F12) → Network tab
- Reload the page
- Right-click the request → "Edit and Resend" (Firefox) or "Copy as fetch" then modify and re-run in the console (Chrome)
- Change the `Referer` header and send

Burp Suite's "Intercept" + "Change header" does the same thing, and is what you'll use for heavier editing later.

## The answer

The password for `natas5` is:

**`iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas4.natas.labs.overthewire.org/>
