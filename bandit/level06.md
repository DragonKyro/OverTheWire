# Bandit Level 6 → 7

## Goal

The password for `bandit7` lives **somewhere on the whole server** (not just your home directory). It's a file that:

- is owned by user `bandit7`
- is owned by group `bandit6`
- is exactly **33 bytes**

You're going to search the entire filesystem for it.

## Concepts you'll learn

- Running `find` from the filesystem root (`/`)
- Filtering `find` results by `-user`, `-group`, and `-size`
- Silencing permission-denied noise with `2>/dev/null`

## Walkthrough

### 1. Log in as bandit6

```bash
$ ssh bandit6@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 5 → 6](level05.md).

### 2. Start `find` at `/`

`/` is the root of the filesystem — the top of everything. Searching from there means "look everywhere."

```bash
$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

What's new here:

- `-user bandit7` — match files owned by the `bandit7` user
- `-group bandit6` — match files whose group owner is `bandit6`
- `-size 33c` — 33 bytes (the `c` means "characters / bytes")
- `2>/dev/null` — throw away everything `find` writes to **standard error**. As `bandit6` walks through `/`, it'll hit thousands of directories it doesn't have permission to read and each one produces a "Permission denied" message on stderr. Redirecting stream 2 to `/dev/null` (the system's bit bucket) suppresses all that noise and leaves only the real results on stdout.

Expected output (a single path):

```
/var/lib/dpkg/info/bandit7.password
```

### 3. Read it

```bash
$ cat /var/lib/dpkg/info/bandit7.password
```

Expected output: one line, the password for `bandit7`.

### 4. Log in as bandit7

```bash
$ ssh bandit7@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit7` is:

**`morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit6.html>
