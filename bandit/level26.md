# Bandit Level 26 → 27

## Goal

You're `bandit26` (via the vi-shell escape from the previous level). In your home is `bandit27-do` — same setuid-helper pattern as `bandit20-do` from [level 19 → 20](level19.md). Use it to read `bandit27`'s password.

## Concepts you'll learn

- Reusing the setuid-helper pattern in a new context
- Confirming what `ls -l` tells you about a binary's owner and its `s` bit
- Building a fresh login session when your current one is a chain of escape hatches (vi → :shell)

## Walkthrough

### 1. Start from the shell you opened in level 25

Continue from the bash shell you spawned out of vi in [level 25 → 26](level25.md). You should already be `bandit26`. Check:

```bash
$ whoami
bandit26
```

### 2. Look at the home directory

```bash
$ ls -l ~
-rwsr-x--- 1 bandit27 bandit26 ... bandit27-do
-rw-r----- 1 bandit26 bandit26 ... text.txt
```

`bandit27-do` has the setuid bit (`rws...`) and is owned by `bandit27`. Same deal as level 19.

### 3. Use it to read the password

```bash
$ ~/bandit27-do cat /etc/bandit_pass/bandit27
```

Expected output: `bandit27`'s password.

> Tip: because you may be in a slightly weird environment (vi's `:shell`, PATH quirks from `/usr/bin/showtext`), use the full path `~/bandit27-do` rather than `./bandit27-do`. You can also inspect `echo $PATH` and `pwd` if anything surprises you.

### 4. Log in as bandit27 in a fresh, normal session

You could keep working from the spawned shell, but a clean login is easier:

```bash
$ ssh bandit27@bandit.labs.overthewire.org -p 2220
```

Use the password you just recovered.

## The answer

The password for `bandit27` is:

**`upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit26.html>
