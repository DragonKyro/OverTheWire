# Bandit Level 19 → 20

## Goal

In `bandit19`'s home is a binary called `bandit20-do`. It's a **setuid** program: it runs with the permissions of its owner (`bandit20`), not the permissions of whoever executes it. Using it, you can run a single command as `bandit20` — including reading `bandit20`'s password file.

## Concepts you'll learn

- What a setuid binary is and how to spot one (the `s` bit in `ls -l`)
- Why setuid is commonly used (and commonly abused)
- Using a helper setuid binary to privilege-escalate to its owner

## Walkthrough

### 1. Log in as bandit19

```bash
$ ssh bandit19@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 18 → 19](level18.md).

### 2. Inspect the binary

```bash
$ ls -l
-rwsr-x--- 1 bandit20 bandit19 14876 ... bandit20-do
```

Read the permissions from left to right:

- `rws` for owner — owner has **r**ead, **w**rite, and the **setuid** bit (the lowercase `s` where you'd expect `x`). The setuid bit means "when this file is executed, the process runs as the file's owner."
- `r-x` for group — group (bandit19) can read and execute
- `---` for others — nobody else

Owner is `bandit20`, so executing this runs as `bandit20`. Exactly what we want.

### 3. Run it with no arguments to see what it does

```bash
$ ./bandit20-do
Run a command as another user.
  Example: ./bandit20-do id
```

It runs whatever command you pass — as `bandit20`.

### 4. Read bandit20's password file

```bash
$ ./bandit20-do cat /etc/bandit_pass/bandit20
```

Expected output: the password for `bandit20`.

As `bandit19`, you couldn't read `/etc/bandit_pass/bandit20` directly (the file is `400 bandit20:bandit20`). Going through `bandit20-do`, you're momentarily running as `bandit20`, which can.

### 5. Log in as bandit20

```bash
$ ssh bandit20@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit20` is:

**`0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit19.html>
