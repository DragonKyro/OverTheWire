# Krypton Level 4 → 5

## Goal

The ciphertext is a **Vigenère cipher** — a polyalphabetic substitution where a short key (like `FREKEY`) cycles through the text and determines a different Caesar shift for every position. The README hints at the key length (**6**), which turns the problem into six independent Caesar ciphers you can solve one at a time.

## Concepts you'll learn

- How Vigenère works: letter `i` of the plaintext is shifted by `key[i mod keylen]`
- The insight that, given the key length, every *n*th character forms a standalone Caesar cipher
- Applying frequency analysis column-by-column to recover the key
- Combining Caesar shifts into the Vigenère key

## Walkthrough

### 1. Log in and read the briefing

```bash
$ ssh krypton4@krypton.labs.overthewire.org -p 2231
$ cd /krypton/krypton4
$ ls
encrypt4  found1  found2  HINT1  krypton5  README
$ cat README
...
The password for krypton5 is in the file 'krypton5'.
Encryption: Vigenère. Key length: 6.
```

Key length is handed to you. Life is easy.

### 2. Understand why key length matters

A Vigenère key of length 6 means:

- positions 0, 6, 12, 18, ... are all shifted by the same letter — **key[0]**
- positions 1, 7, 13, 19, ... are all shifted by **key[1]**
- ... and so on.

So if you pull out **every 6th character** into a separate string, that string is a simple Caesar cipher: just one shift to figure out. Do it six times (once per key position) and you have the whole key.

### 3. Extract columns and do frequency analysis on each

Grab the target ciphertext:

```bash
$ cat krypton5/krypton5
# (one line of uppercase letters, possibly with spaces removed)
```

Python script to break the Vigenère with known key length:

```python
#!/usr/bin/env python3
from collections import Counter

ct = open('/krypton/krypton4/krypton5/krypton5').read().upper()
ct = ''.join(c for c in ct if c.isalpha())  # keep only letters
KEYLEN = 6

# For each column, find the Caesar shift that makes E the most common letter.
key = []
for i in range(KEYLEN):
    column = ct[i::KEYLEN]                       # every KEYLEN-th char starting at i
    most_common = Counter(column).most_common(1)[0][0]
    # Assume most_common in ct maps to E in pt.
    shift = (ord(most_common) - ord('E')) % 26
    key.append(chr(shift + ord('A')))

print('Key:', ''.join(key))

# Decrypt with the recovered key.
out = []
for i, c in enumerate(ct):
    k = ord(key[i % KEYLEN]) - ord('A')
    out.append(chr((ord(c) - ord('A') - k) % 26 + ord('A')))
print(''.join(out))
```

Save and run:

```bash
$ python3 /tmp/vig.py
Key: FREKEY
THEPASSWORDFORLEVELFIVEISCLEARTEXT
```

`FREKEY` was the key; the plaintext reveals `CLEARTEXT` as the level-5 password.

### 4. If frequency alone doesn't land it

For short ciphertexts, the most common letter in a column might not be `E` — it could be `T` or `A` depending on the sample. If the output looks like mostly-nonsense English with a few plausible words, try the next most common shifts. In the Python above, replace the `most_common(1)` with `most_common(5)` and inspect; a human can usually spot which shift produced the English-iest column.

### 5. Log in as krypton5

```bash
$ ssh krypton5@krypton.labs.overthewire.org -p 2231
```

## The answer

The password for `krypton5` is:

**`CLEARTEXT`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/krypton/krypton4.html>
