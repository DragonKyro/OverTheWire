# Krypton Level 5 → 6

## Goal

Same Vigenère cipher as the previous level — but this time **the key length is not given**. You have to figure it out before you can split the ciphertext into columns and attack it.

## Concepts you'll learn

- The **Kasiski examination** and the **index of coincidence** — two classical methods for estimating Vigenère key length
- A brute-force alternative: try key lengths 1, 2, 3, … and keep the one that produces English
- Why long ciphertexts make both approaches far more reliable than short ones

## Walkthrough

### 1. Log in and look at the files

```bash
$ ssh krypton5@krypton.labs.overthewire.org -p 2231
$ cd /krypton/krypton5
$ ls
encrypt5  found1  found2  HINT1  krypton6  README
$ cat README
# Same Vigenère cipher as before, but we're not telling you the key length.
```

### 2. The conceptually-right approach: Kasiski examination

Kasiski's trick: if a word (a trigram or longer) appears twice in the plaintext *aligned with the same key position*, it encrypts to the same ciphertext both times. So **repeated ciphertext substrings** hint at plaintext repetitions, and the **distance between those repetitions** is a multiple of the key length.

If you find repetitions at distances 24, 36, 48, the GCD is 12 → key length divides 12. Candidates: 12, 6, 4, 3, 2.

A short Python script for Kasiski:

```python
#!/usr/bin/env python3
from math import gcd
from functools import reduce

ct = open('/krypton/krypton5/krypton6/krypton6').read()
ct = ''.join(c for c in ct.upper() if c.isalpha())

positions = {}
for i in range(len(ct) - 2):
    tri = ct[i:i+3]
    positions.setdefault(tri, []).append(i)

distances = []
for tri, pos_list in positions.items():
    if len(pos_list) < 2:
        continue
    for j in range(1, len(pos_list)):
        distances.append(pos_list[j] - pos_list[0])

if distances:
    g = reduce(gcd, distances)
    print(f'GCD of repeat distances: {g}')
    print('Likely key lengths:', [n for n in range(2, 16) if g % n == 0])
```

For this level you'll typically land on a key length of **9** (the key turns out to be `KEYLENGTH`, 9 letters).

### 3. The pragmatic approach: try key lengths until one works

If Kasiski feels heavy, wrap the level-4 script in an outer loop:

```python
#!/usr/bin/env python3
from collections import Counter

ct = open('/krypton/krypton5/krypton6/krypton6').read().upper()
ct = ''.join(c for c in ct if c.isalpha())

def try_length(keylen):
    key = []
    for i in range(keylen):
        col = ct[i::keylen]
        top = Counter(col).most_common(1)[0][0]
        shift = (ord(top) - ord('E')) % 26
        key.append(chr(shift + ord('A')))
    key = ''.join(key)
    out = ''.join(
        chr((ord(c) - ord('A') - (ord(key[i % keylen]) - ord('A'))) % 26 + ord('A'))
        for i, c in enumerate(ct)
    )
    return key, out

for L in range(2, 13):
    key, pt = try_length(L)
    print(f'L={L:2d} key={key}')
    print('   ', pt[:80])
    print()
```

Run it and scan the outputs. When `L` hits the real key length, the plaintext suddenly becomes readable English. Axcheron landed on key **`KEYLENGTH`** (9 letters); the decrypted plaintext reveals:

```
THEPASSWORDFORLEVELSIXISRANDOM
```

### 4. Log in as krypton6

```bash
$ ssh krypton6@krypton.labs.overthewire.org -p 2231
```

Use `RANDOM` as the password.

## The answer

The password for `krypton6` is:

**`RANDOM`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/krypton/krypton5.html>
