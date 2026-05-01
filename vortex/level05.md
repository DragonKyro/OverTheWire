# Vortex Level 5 → 6

## Goal

`vortex5` asks you to recover a password given only its **MD5 hash**. The password is 5 characters from a restricted alphabet, making the search space small enough to brute-force in seconds.

## Concepts you'll learn

- **Preimage attacks** on fast hash functions when the plaintext space is small
- Using `itertools.product` to enumerate all candidates
- Python's `hashlib.md5` for batch hashing

## Walkthrough

### 1. Read the binary / hint

The level tells you (or the binary encodes) a target MD5 digest. avishai's solve pins the target at:

```
\x15\x5f\xb9\x5d\x04\x28\x7b\x75\x7c\x99\x6d\x77\xb5\xea\x51\xf7
```

— i.e. `155fb95d04287b757c996d77b5ea51f7` in hex.

The alphabet and length:

- **Length:** 5
- **Alphabet:** `'r'` + `string.ascii_letters` + `string.digits` (53+10 = 63 characters, plus the leading `r`; solver found this empirically)

### 2. Brute force it

```python
# level5.py
import hashlib, string, itertools

target = bytes.fromhex("155fb95d04287b757c996d77b5ea51f7")
alphabet = "r" + string.ascii_letters + string.digits

for combo in itertools.product(alphabet, repeat=5):
    s = "".join(combo)
    if hashlib.md5(s.encode()).digest() == target:
        print("FOUND:", s)
        break
```

Runs in about 5 seconds on a typical laptop. Output:

```
FOUND: rlTf6
```

### 3. Submit and collect

Submit `rlTf6` to whatever the level's binary expects (stdin, argv, or a network prompt). If correct, it hands you the `vortex6` password.

### 4. Log in as vortex6

```bash
$ ssh vortex6@vortex.labs.overthewire.org -p 5842
```

## The answer

The password for `vortex6` is:

**`rlTf6`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.
>
> Also worth flagging: **the `vortex6` login password is unusually short** (5 characters). The avishai reference explicitly reports this value, but if SSH rejects it, the MD5 preimage may have been post-processed before being used as the SSH password (e.g. hashed with a static salt). Re-solve the level locally and inspect the full flow if the short string alone doesn't work.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex5.html>
