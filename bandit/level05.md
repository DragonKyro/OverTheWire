# Bandit Level 5 → 6

## Goal

The password for `bandit6` is in `inhere/`, buried in a deep directory tree. The file you want has three specific properties:

- human-readable
- exactly **1033 bytes** in size
- **not executable**

Your job is to search the tree and find the one file matching all three.

## Concepts you'll learn

- Using `find` to search a directory tree
- `find` filters: `-type`, `-size`, `-readable`, `! -executable`
- Reading the results of `find` and feeding one into `cat`

## Walkthrough

### 1. Log in as bandit5

```bash
$ ssh bandit5@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 4 → 5](level04.md).

### 2. Peek at the structure

```bash
$ ls
```

Expected output:

```
inhere
```

```bash
$ ls inhere
```

Expected output (a bunch of subdirectories):

```
maybehere00  maybehere01  maybehere02  ...  maybehere19
```

Each of those holds more files. Too many to check by hand — that's exactly what `find` is for.

### 3. Use `find` with the right filters

```bash
$ find inhere -type f -size 1033c ! -executable
```

Breaking that down:

- `find inhere` — start searching inside the `inhere` directory and descend into every subdirectory
- `-type f` — only match regular files (not directories)
- `-size 1033c` — match files of size 1033 **c**haracters (bytes). Without the `c` it would mean 1033 *512-byte blocks*, which is very different.
- `! -executable` — exclude files the current user could execute. `!` means "not."

Expected output (a single path):

```
inhere/maybehere07/.file2
```

(The exact `maybehere##` directory may differ between solves.)

### 4. Read it

```bash
$ cat inhere/maybehere07/.file2
```

Expected output: one line, the password for `bandit6`.

> Tip: you can combine the two steps into one with command substitution: `cat "$(find inhere -type f -size 1033c ! -executable)"`. That's handy once you trust `find` to return exactly one result.

### 5. Log in as bandit6

```bash
$ ssh bandit6@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit6` is:

**`HWasnPhtq9AVKe0dmk45nxy20cvUa6EG`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit5.html>
