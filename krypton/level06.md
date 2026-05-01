# Krypton Level 6 → 7

## Goal

This is the final Krypton level. The ciphertext was made with a **stream cipher** — plaintext XORed against a keystream produced by a weak pseudo-random generator. You have the `encrypt6` binary, so you can mount a **known-plaintext attack**: feed in something you know (like a long run of `A`s), capture the output, XOR to recover the keystream, and use that keystream to decrypt the real password file.

## Concepts you'll learn

- What a stream cipher is: `ciphertext[i] = plaintext[i] XOR keystream[i]`
- Why XOR is reversible — if you know any two of the three (plaintext, ciphertext, keystream) you know the third
- A **known-plaintext attack**: derive the keystream when you control the plaintext input
- Spotting **keystream reuse / short period** in a weak PRNG

## Walkthrough

### 1. Log in and look at the files

```bash
$ ssh krypton6@krypton.labs.overthewire.org -p 2231
$ cd /krypton/krypton6
$ ls
encrypt6  keyfile.dat  krypton7  README
$ cat README
# encrypt6 is a stream-cipher encryptor. The keystream comes from keyfile.dat.
# krypton7/krypton7 contains the encrypted password for krypton7.
```

The `krypton7` *directory* contains the target:

```bash
$ cat krypton7/krypton7
PNUKLYLWRQKGKBE
```

And `encrypt6` is the encryptor you can feed with whatever plaintext you want.

### 2. Produce a known keystream

XOR of anything with a zero-valued byte is that byte itself. But plain text alphabets are ASCII, not null, so we'll use a block of repeating `A`s and derive the keystream from the relationship `K[i] = C[i] XOR 'A'`.

```bash
$ mkdir /tmp/t6
$ cd /tmp/t6
$ python3 -c "print('A'*50)" > plain.txt
$ /krypton/krypton6/encrypt6 plain.txt cipher.txt
$ cat cipher.txt
```

`encrypt6` takes `<infile> <outfile>` (or reads stdin — check the binary's usage). The ciphertext will be something like `EICTDGYIYZKTHNSIRFXYCPFUEOCKRN` — the same 30 characters repeating, because the keystream has a period of 30 and you gave it 50 characters of identical input.

### 3. Derive the keystream

Because the plaintext is all `A`s, the keystream equals ciphertext shifted back by `A`:

```python
#!/usr/bin/env python3
cipher = open('/tmp/t6/cipher.txt').read().strip()
keystream = [(ord(c) - ord('A')) % 26 for c in cipher]
print(keystream[:30])   # one full period
```

(This treats "XOR" as the level's arithmetic of choice — some Krypton versions use modular addition rather than bitwise XOR. If bitwise XOR, substitute `ord(c) ^ ord('A')` and work with 8-bit values.)

Axcheron's keystream for one such solve was:

```
EICTDGYIYZKTHNSIRFXYCPFUEOCKRN
```

### 4. Use the keystream to decrypt the target

```python
#!/usr/bin/env python3
ct = open('/krypton/krypton6/krypton7/krypton7').read().strip()
key = "EICTDGYIYZKTHNSIRFXYCPFUEOCKRN"
out = []
for i, c in enumerate(ct):
    k = ord(key[i % len(key)]) - ord('A')
    out.append(chr((ord(c) - ord('A') - k) % 26 + ord('A')))
print(''.join(out))
```

Expected output:

```
LFSRISNOTRANDOM
```

(`LFSR` is a callback to Linear-Feedback Shift Registers — the classic simple PRNG this level's generator is meant to echo.)

### 5. Log in as krypton7 — the end of the game

```bash
$ ssh krypton7@krypton.labs.overthewire.org -p 2231
```

You've reached the final user. As `krypton7`, there are no more ciphertexts — you've completed Krypton. From here, OTW's natural next stops are [Narnia](../narnia/README.md) or [Behemoth](../behemoth/README.md) for binary exploitation, or another web-focused game like [Natas](../natas/README.md).

## The answer

The password for `krypton7` is:

**`LFSRISNOTRANDOM`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/krypton/krypton6.html>
