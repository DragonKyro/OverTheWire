# Natas Level 5 → 6

## Goal

The page says *"Access disallowed. You are not logged in."* The server decides whether you're logged in based on a **cookie** called `loggedin` — which is set to `0`. You set it to `1`.

## Concepts you'll learn

- HTTP cookies: small key-value pairs your browser sends to the server with every request
- Using browser dev tools to view and edit cookies, or `curl -b` to send a custom cookie
- That *client-side state* (cookies, hidden form fields, JS variables) must **never** be trusted for access decisions

## Walkthrough

### 1. Inspect the cookie

Open `http://natas5.natas.labs.overthewire.org/` as `natas5`. In dev tools (F12) → **Application** (Chrome) or **Storage** (Firefox) tab → **Cookies** → the Natas 5 domain. You'll see:

```
Name       Value
loggedin   0
```

### 2. Flip it to 1

Click the value, change `0` to `1`, Enter. Reload the page. The "Access disallowed" message is gone and the password is shown.

### 3. (Alternative) curl

```bash
$ curl -u natas5:iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq \
       -b "loggedin=1" \
       http://natas5.natas.labs.overthewire.org/
```

`-b "cookie=value"` attaches a cookie header.

Expected output contains:

```
The password for natas6 is aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1
```

## The answer

The password for `natas6` is:

**`aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas5.natas.labs.overthewire.org/>
