# Natas Level 22 → 22

## Goal

When you open the page, the server uses `header("Location: ...")` to redirect you away before it prints the password. **BUT** — `header()` in PHP doesn't stop execution by itself, and the server only actually sends the redirect if you don't disable it. Pass a specific query string (`?revelio=1`) and the redirect branch is skipped; the password rendering runs and is sent to you.

## Concepts you'll learn

- `header("Location: ...")` in PHP **sets** a redirect header but does **not** terminate the script
- The importance of reading source before assuming a redirect means "that's the end of it"
- Inspecting the **response body** even when the status is a 302 — with `curl -i` or dev tools' Network tab

## Walkthrough

### 1. View the source

```php
session_start();

if(array_key_exists("revelio", $_GET)) {
    // ...
} else {
    header("Location: /");
}

if($_SESSION["admin"] == 1) {
    echo "The password for natas23 is <censored>";
}
```

Reading carefully:

- Without `?revelio=1`, the `header("Location: /")` runs.
- `header()` does *not* stop execution — subsequent code still runs.
- Then the `$_SESSION["admin"]` check runs. If `admin == 1`, the password is echoed into the response body (which is still sent, even alongside the redirect header).

But even without worrying about the admin flag: with `?revelio=1`, the redirect is skipped entirely, and if the session happens to allow admin, the password prints.

### 2. Hit the URL with ?revelio=1

```bash
$ curl -u natas22:<pass> http://natas22.natas.labs.overthewire.org/index.php?revelio=1
```

If you get `admin=1` in your session (or the default does), the output includes:

```
D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE
```

If not, look back at level 21's trick to set `admin=1` via the experimenter site. Some iterations of this level arrange the session for you; others require it. Try `?revelio=1` alone first.

### 3. Browser alternative

In the browser, just navigate to:

```
http://natas22.natas.labs.overthewire.org/index.php?revelio=1
```

The browser honours the `Location` header and redirects — so use dev tools' Network tab: disable redirects, or inspect the first response's body (the password is there even though the browser auto-redirects away from it).

## The answer

The password for `natas23` is:

**`D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas22.natas.labs.overthewire.org/>
