# Krypton Level 3 → 4

## Goal

The ciphertext for `krypton4`'s password was made with a **monoalphabetic substitution cipher**: each letter of the alphabet is mapped to some other letter, consistently throughout the text. The key isn't a simple shift, so you can't brute-force 26 possibilities — you need **letter-frequency analysis** to recover the mapping.

## Concepts you'll learn

- English letter frequency: `E T A O I N S H R D L C U M W F G Y P B V K J X Q Z` (most → least common)
- How three **sample plaintexts encrypted with the same key** turn "mystery cipher" into a solvable statistics problem
- Counting letter frequencies in Python and matching them against English
- Iterating on your candidate mapping: standard frequency is a starting guess, not a perfect answer

## Walkthrough

### 1. Log in and look at the files

```bash
$ ssh krypton3@krypton.labs.overthewire.org -p 2231
$ cd /krypton/krypton3
$ ls
found1  found2  found3  HINT1  HINT2  krypton4  README
```

- `found1`, `found2`, `found3` — three ciphertext files that were "found" during transmission. All encrypted with the same unknown key.
- `krypton4` — a directory containing the actual target ciphertext
- `HINT1`, `HINT2` — level hints

Read the hints:

```bash
$ cat HINT1
Run a letter frequency analysis on the files. Compare their frequency to English.

$ cat HINT2
Use the KNOWN plaintext files to build your substitution table...
```

### 2. Count letter frequencies on all three samples

Combine the files and tally letter counts:

```bash
$ cat found1 found2 found3 > /tmp/all_ct
$ python3 <<'EOF'
from collections import Counter
text = open('/tmp/all_ct').read().upper()
letters = [c for c in text if c.isalpha()]
freq = Counter(letters).most_common()
for letter, count in freq:
    print(f'{letter}: {count}')
EOF
```

Expected output (order approximate):

```
Q: 342
T: 288
S: 215
O: 201
R: 198
I: 167
N: 152
H: 134
...
```

(Your counts will vary, but the *ordering* matters — the most common ciphertext letter very likely corresponds to English `E`, the second most common to `T`, and so on.)

### 3. Build an initial mapping and decrypt

Use the English frequency table `ETAOINSHRDLCUMWFGYPBVKJXQZ` as a starting guess. If your ciphertext frequency order is `QTSORINHC...`, then:

```
cipher letter: Q T S O R I N H C ...
plaintext guess: E T A O I N S H R ...
```

A Python script to translate:

```python
#!/usr/bin/env python3
ct_freq = "QTSORINHCLDUPMFWGYBKVXQJZ"  # your observed order; adjust!
en_freq = "ETAOINSHRDLCUMWFGYPBVKJXQZ"
mapping = str.maketrans(ct_freq, en_freq)

target = open('/krypton/krypton3/krypton4/krypton4').read()
print(target.translate(mapping))
```

Run it:

```bash
$ python3 /tmp/decrypt.py
ZAONGF YMSOFU PEGZ TSSTMY...
```

### 4. Iterate — the first guess is rarely perfect

English frequency is only approximate. Common letters like `E`, `T`, `A`, `O` usually land correctly; rarer letters often swap around. Look at the partial output, spot English-looking words with one or two wrong letters, and swap pairs of letters in your mapping until the output makes sense.

For example, if you see the word `BRUTF` where `BRUTE` would fit, then `F` in your output should be `E`, so the letter currently mapped to `F` is actually `E` — rotate your mapping and re-run.

Axcheron's final mapping landed at:

```
ct: EQTSORINHCLDUPMFWGYBKVXQJZ
pt: ETAOINSHRDLCUMWFGYPBVKJXQZ
```

With the correct mapping, the decryption yields:

```
THE PASSWORD IS BRUTE
```

### 5. Log in as krypton4

```bash
$ ssh krypton4@krypton.labs.overthewire.org -p 2231
```

## The answer

The password for `krypton4` is:

**`BRUTE`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/krypton/krypton3.html>
