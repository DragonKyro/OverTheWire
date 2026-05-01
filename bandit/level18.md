# Bandit Level 18 → 19

## Goal

You have `bandit18`'s password, but the moment you SSH in, you're immediately booted back out — someone has modified `bandit18`'s `~/.bashrc` to log you out. The password for `bandit19` is in a `readme` file in `bandit18`'s home directory, and you need a way to read it without letting `.bashrc` run.

## Concepts you'll learn

- What `~/.bashrc` is: a shell startup script that runs every time you open an interactive bash session
- Running a **single command** over SSH instead of an interactive shell — which skips `.bashrc` (or at least the interactive portion)
- Using alternative shells (`bash --noprofile --norc`, `/bin/sh`) to sidestep a hostile `.bashrc`

## Walkthrough

### 1. See the problem

```bash
$ ssh bandit18@bandit.labs.overthewire.org -p 2220
```

After you enter the password, you see something like:

```
Byebye !
Connection to bandit.labs.overthewire.org closed.
```

The session died instantly. The `.bashrc` fired an `exit`.

### 2. Run a one-shot command instead of opening a shell

If you append a command to your `ssh` invocation, SSH runs that command on the remote instead of giving you an interactive shell:

```bash
$ ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```

When prompted, enter the password. Expected output:

```
<PASSWORD for bandit19>
```

`cat readme` ran in a non-interactive shell — the hostile `.bashrc` either didn't run its `exit` path (it checks for interactive mode first) or didn't have time to matter before `cat` finished.

### 3. (Alternative) Ask for a different shell

If the non-interactive trick didn't work for some reason, you can force SSH to start a shell that doesn't read `.bashrc`:

```bash
$ ssh bandit18@bandit.labs.overthewire.org -p 2220 -t "bash --noprofile --norc"
```

`--noprofile` skips `/etc/profile` and friends; `--norc` skips `.bashrc`. `-t` forces a pseudo-terminal so the shell feels interactive.

Or even simpler:

```bash
$ ssh bandit18@bandit.labs.overthewire.org -p 2220 -t /bin/sh
```

`sh` doesn't read `.bashrc` at all. Once you're in, `cat readme`.

### 4. Log in as bandit19

```bash
$ ssh bandit19@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit19` is:

**`cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit18.html>
