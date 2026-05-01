# Krypton Level 1 → 2

## Goal

In `/krypton/krypton1` you'll find a ciphertext encrypted with **ROT13** — a fixed-shift Caesar cipher where every letter is rotated 13 positions through the alphabet. Decrypt it to recover the next password.

## Concepts you'll learn

- The Caesar cipher family: shift every letter by a constant number of positions
- Why ROT13 is self-inverse (13 + 13 = 26, a full alphabet cycle) — the same program that encrypts also decrypts
- Using `tr` for ROT13 in a single shell command

## Walkthrough

### 1. Log in and look around

```bash
$ ssh krypton1@krypton.labs.overthewire.org -p 2231
$ cd /krypton/krypton1
$ ls
krypton2  README
```

Read the README for context:

```bash
$ cat README
Welcome to Krypton!
...
The password for level 2 is in the file 'krypton2'.
It is ROT13 encrypted.
```

And look at the ciphertext:

```bash
$ cat krypton2
YRIRY GJB CNFFJBEQ EBGGRA
```

Looks gibberish — uppercase English letters and spaces. That shape screams "substitution cipher." The README tells us it's ROT13.

### 2. Decrypt with `tr`

```bash
$ cat krypton2 | tr 'A-Za-z' 'N-ZA-Mn-za-m'
LEVEL TWO PASSWORD ROTTEN
```

Same one-liner you saw in [Bandit 11 → 12](../bandit/level11.md): `tr` translates every character in the first set to the corresponding character in the second set, where the second set is the first set shifted 13 positions.

The plaintext tells you the `krypton2` password is `ROTTEN`.

### 3. Log in as krypton2

```bash
$ ssh krypton2@krypton.labs.overthewire.org -p 2231
```

## The answer

The password for `krypton2` is:

**`ROTTEN`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/krypton/krypton1.html>
