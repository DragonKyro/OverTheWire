# Bandit Level 10 → 11

## Goal

The password for `bandit11` is inside `data.txt`, but it's been encoded with **base64**. You need to recognise the encoding and decode it.

## Concepts you'll learn

- What base64 is: a way to represent arbitrary bytes using only 64 printable ASCII characters (`A-Z`, `a-z`, `0-9`, `+`, `/`), with `=` as padding
- That base64 is **encoding**, not encryption — anybody with the tool can decode it
- Using the `base64 -d` command to decode

## Walkthrough

### 1. Log in as bandit10

```bash
$ ssh bandit10@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 9 → 10](level09.md).

### 2. Look at the file

```bash
$ cat data.txt
```

Expected output:

```
VGhlIHBhc3N3b3JkIGlzIGR0UjE3M2ZaS2IwUlJzREZTR3NnMlJXbnBOVmozcVJyCg==
```

A couple of tells that this is base64:

- Only A-Z, a-z, 0-9, `+`, `/` characters
- Ends in one or two `=` signs (padding)

### 3. Decode it

```bash
$ base64 -d data.txt
```

`-d` means "decode." Expected output:

```
The password is dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr
```

The password for `bandit11` follows "is ".

### 4. Log in as bandit11

```bash
$ ssh bandit11@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit11` is:

**`dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit10.html>
