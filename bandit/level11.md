# Bandit Level 11 → 12

## Goal

The password for `bandit12` is in `data.txt`, encoded with **ROT13** — a classical cipher that rotates each letter 13 positions through the alphabet. You need to apply the same transformation again to reverse it (because 13 + 13 = 26 = a full cycle).

## Concepts you'll learn

- What ROT13 is and why it's self-inverse
- Using `tr` to translate one set of characters to another
- Writing a character-range mapping for `tr`

## Walkthrough

### 1. Log in as bandit11

```bash
$ ssh bandit11@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 10 → 11](level10.md).

### 2. Look at the file

```bash
$ cat data.txt
```

Expected output (something like):

```
Gur cnffjbeq vf 7k16JArHIi5YxIhWsfFIqoogmTYlvgLM
```

Clearly English-ish in shape but scrambled. That's ROT13.

### 3. Apply ROT13 with `tr`

`tr` (translate) reads from stdin and swaps characters from one set to another. For ROT13 you want:

- `A` → `N`, `B` → `O`, ..., `M` → `Z`
- `N` → `A`, ..., `Z` → `M`
- Same for lowercase

```bash
$ cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

Breaking it down:

- `A-Za-z` — the *from* set: every uppercase letter, then every lowercase letter (52 chars)
- `N-ZA-Mn-za-m` — the *to* set: the letters 13 positions later, wrapping around

Expected output:

```
The password is 7x16WNeHIi5YkIhWsfFIqoognSkmuLhg
```

The password follows "is ".

### 4. Log in as bandit12

```bash
$ ssh bandit12@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit12` is:

**`7x16WNeHIi5YkIhWsfFIqoognSkmuLhg`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit11.html>
