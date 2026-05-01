# Krypton Level 0 → 1

## Goal

Unlike every other OverTheWire wargame, Krypton doesn't hand you an SSH login with matching credentials. Instead, the level-0 "password" is given to you **on the Krypton page** as a base64-encoded blob. Decode it to get the real SSH password for `krypton1`.

## Concepts you'll learn

- What base64 is — a way of encoding arbitrary bytes using a 64-character alphabet (`A-Z`, `a-z`, `0-9`, `+`, `/`)
- That base64 is **encoding, not encryption** — anybody with a decoder can read it
- The difference between *encoding* (reversible without a secret) and *encryption* (requires a key)

## Walkthrough

### 1. Grab the blob from the OTW page

The OverTheWire level-0 page contains this string:

```
S1JZUFRPTklTR1JFQVQ=
```

Key tells that this is base64:

- Only `A-Z`, `a-z`, `0-9`, `+`, `/` characters
- Ends with `=` padding
- Length is a multiple of 4

### 2. Decode it

On any Linux/macOS machine:

```bash
$ echo 'S1JZUFRPTklTR1JFQVQ=' | base64 -d
KRYPTONISGREAT
```

`base64 -d` means "decode." The output is the password for `krypton1`.

If you'd rather do it in Python (since later levels will want some scripting):

```python
>>> import base64
>>> base64.b64decode(b'S1JZUFRPTklTR1JFQVQ=')
b'KRYPTONISGREAT'
```

### 3. SSH in as krypton1

```bash
$ ssh krypton1@krypton.labs.overthewire.org -p 2231
```

Use `KRYPTONISGREAT` as the password.

## The answer

The password for `krypton1` is:

**`KRYPTONISGREAT`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/krypton/krypton0.html>
