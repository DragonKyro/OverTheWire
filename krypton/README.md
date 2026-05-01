# Krypton

> A hands-on tour of classical cryptography — from Caesar shifts to frequency analysis to attacks on misused modern ciphers.

## About the game

Krypton walks you through the history of cryptography one broken cipher at a time. You don't need to be a mathematician; you need to be willing to spot patterns, count letters, and read the small programs the level authors provide. Each level gives you one or more ciphertexts and (usually) the encrypting tool, and your job is to recover the plaintext — which contains the next level's password.

**Difficulty:** Beginner to intermediate, depending on level. Later levels introduce real cryptanalysis concepts.

## Purpose

Krypton's goal is to demystify cryptography. By the end you've actually broken Caesar, Vigenère, and a handful of stream/block-cipher misuses, which makes "crypto" feel like something you can reason about — not a black box the textbook told you was secure.

## What you'll learn

- The difference between encoding (base64, hex) and encryption
- Breaking monoalphabetic substitution with frequency analysis
- Attacking the Vigenère cipher (Kasiski / index-of-coincidence)
- One-time pad vs "many-time pad" — why key reuse is catastrophic
- Recognising bad modes of operation (ECB patterns, predictable IVs)
- Writing short Python (or shell) scripts to automate classical attacks

## How to connect

Krypton is unusual: the level-0 "password" is given to you on the website as a base64 blob rather than the usual matching-username form. Decode it, then SSH in as `krypton1` straight away.

```bash
# Decode the level-0 password
echo 'S1JZUFRPTklTR1JFQVQ=' | base64 -d
# -> KRYPTONISGREAT

# Connect as krypton1 using that password
ssh krypton1@krypton.labs.overthewire.org -p 2231
```

- **Host:** `krypton.labs.overthewire.org`
- **Port:** `2231`
- **Starting username:** `krypton1`
- **Starting password:** `KRYPTONISGREAT` (base64-decoded from the value on the OTW site)

## Level index

| Level | Writeup |
|------:|---------|
| 0 → 1 | [🔒 Not yet written](level00.md) |
| 1 → 2 | [🔒 Not yet written](level01.md) |
| 2 → 3 | [🔒 Not yet written](level02.md) |
| 3 → 4 | [🔒 Not yet written](level03.md) |
| 4 → 5 | [🔒 Not yet written](level04.md) |
| 5 → 6 | [🔒 Not yet written](level05.md) |
| 6 → 7 | [🔒 Not yet written](level06.md) |

## Disclaimer

OverTheWire may rotate passwords over time. Published passwords in per-level writeups are a snapshot from solve time — if a password fails, re-solve using the same method on the current ciphertext.

## References

- Official game page: <https://overthewire.org/wargames/krypton/>
