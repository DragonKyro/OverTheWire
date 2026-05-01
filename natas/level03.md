# Natas Level 3 → 4

## Goal

The page says "No more information leaks!! Not even Google will find it this time..." — which is a hint. Check `robots.txt` (the file that tells search engines what to skip) and follow the trail.

## Concepts you'll learn

- What `robots.txt` is and why it's *not* a security control
- That `Disallow:` lines in `robots.txt` tell *attackers* exactly where to look
- The same directory-listing trick as the previous level

## Walkthrough

### 1. Log in and view the source

`http://natas3.natas.labs.overthewire.org/` with user `natas3` and the last level's password.

View source — nothing useful this time. The hint "Not even Google" points at `robots.txt`.

### 2. Fetch robots.txt

```
http://natas3.natas.labs.overthewire.org/robots.txt
```

Contents:

```
User-agent: *
Disallow: /s3cr3t/
```

Directory `/s3cr3t/` is being "hidden" from search engines. It's not actually hidden from you.

### 3. Browse the disallowed directory

```
http://natas3.natas.labs.overthewire.org/s3cr3t/
```

Directory listing again:

```
Index of /s3cr3t
  ../
  users.txt
```

### 4. Read users.txt

```
http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt
```

Contents:

```
natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ
```

## The answer

The password for `natas4` is:

**`Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas3.natas.labs.overthewire.org/>
