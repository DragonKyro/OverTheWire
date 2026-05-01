# Natas Level 21 → 22

## Goal

Natas 21 has a sibling site at `natas21-experimenter.natas.labs.overthewire.org` that shares its PHP session storage with the main site. The experimenter site lets you set arbitrary key-value pairs in **your own session** (including `admin=1`). You set the value there, then browse to the main site — same session, same cookie, admin flag carries over.

## Concepts you'll learn

- **Shared session storage** between subdomains — a consequence of shared cookies, PHPSESSID reuse, or explicit session-save-handler configuration
- Looking at a web app and asking "what other endpoints on this host write to my session?"
- Moving a cookie between origins when those origins cooperate

## Walkthrough

### 1. Explore the experimenter site

Open:

```
http://natas21-experimenter.natas.labs.overthewire.org/
```

Use the same credentials (`natas21` / the previous password — HTTP Basic works across both subdomains because they share the same realm). You'll see a form that writes whatever keys you submit into `$_SESSION`. Source:

```php
foreach($_REQUEST as $key => $val) {
    $_SESSION[$key] = $val;
}
```

No filtering. Submitting `admin=1` writes `$_SESSION['admin'] = 1`.

### 2. Grab the shared PHPSESSID

The experimenter site sets a `PHPSESSID` cookie. Because the main Natas 21 site is configured to use the **same session storage**, that same session ID is valid on both sides.

Submit `admin=1` on the experimenter site, then copy the `PHPSESSID` cookie value.

### 3. Use the cookie on the main site

In your browser, switch over to `http://natas21.natas.labs.overthewire.org/`, edit the `PHPSESSID` cookie to the value from step 2, reload. Or with curl:

```bash
$ curl -u natas21:<pass> -b "PHPSESSID=<id from experimenter>" \
       http://natas21.natas.labs.overthewire.org/index.php
```

The main site now reads your session, sees `admin=1`, and prints:

```
chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ
```

## The answer

The password for `natas22` is:

**`chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas21.natas.labs.overthewire.org/>
