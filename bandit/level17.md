# Bandit Level 17 → 18

## Goal

`bandit17`'s home directory contains two files, `passwords.old` and `passwords.new`. They're almost identical — **one line differs**, and that differing line in `passwords.new` is the `bandit18` password.

## Concepts you'll learn

- Using `diff` to compare two text files line-by-line
- Reading `diff`'s default "normal" output format
- Scoping a diff to just the different lines (skipping the identical ones)

## Walkthrough

### 1. Log in as bandit17

```bash
$ ssh bandit17@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 16 → 17](level16.md).

### 2. Look at the files

```bash
$ ls
passwords.new  passwords.old
$ wc -l passwords.new passwords.old
```

Both files are the same number of lines. You want the single difference.

### 3. Diff them

```bash
$ diff passwords.new passwords.old
```

Expected output (something like):

```
42c42
< x2gLTTjFz5gHQZPk0qblncTNDX8UNq5b
---
> oldlinefrompasswordsold
```

Reading a default diff:

- `42c42` — "line 42 was **c**hanged"; there's one line on each side
- `<` — the line from the first file (`passwords.new`)
- `---` — separator
- `>` — the line from the second file (`passwords.old`)

The line after `<` is from `passwords.new` — that's your new password.

### 4. Log in as bandit18

```bash
$ ssh bandit18@bandit.labs.overthewire.org -p 2220
```

Before you try: look at [level 18 → 19](level18.md) first — `bandit18`'s shell has a surprise waiting for you.

## The answer

The password for `bandit18` is:

**`x2gLTTjFz5gHQZPk0qblncTNDX8UNq5b`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit17.html>
