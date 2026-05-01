# Bandit Level 28 → 29

## Goal

Clone `bandit28`'s git repo. The README looks like it has the password redacted — but the **history** of the repo still shows the password that was committed before someone thought to hide it.

## Concepts you'll learn

- `git log` — viewing the commit history
- `git log -p` / `git show <commit>` — seeing the actual diff that each commit introduced
- The reality that removing a secret from a file in a new commit does not remove it from history

## Walkthrough

### 1. Log in as bandit28

```bash
$ ssh bandit28@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 27 → 28](level27.md).

### 2. Clone the repo

```bash
$ cd /tmp
$ mkdir mygit28 && cd mygit28
$ git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repo
$ cd repo
```

Supply your `bandit28` password when prompted.

### 3. Look at the README

```bash
$ cat README.md
```

Expected output (something like):

```
# Bandit Notes
Some notes for level29 of bandit.

## credentials
- username: bandit29
- password: xxxxxxxxxx
```

Someone redacted the password to `xxxxxxxxxx`. But they committed the real password first.

### 4. Look at the history

```bash
$ git log --oneline
```

Expected output (a short list of commits):

```
abc1234 fix info leak
def5678 add some data here
1234567 initial commit
```

The commit message "fix info leak" is a big hint. Look at its diff:

```bash
$ git log -p
```

Or more precisely, show that one commit:

```bash
$ git show abc1234
```

Expected output (among other lines):

```
- password: <THE REAL PASSWORD>
+ password: xxxxxxxxxx
```

The line starting with `-` is what was removed: the original, real password.

### 5. Log in as bandit29

```bash
$ ssh bandit29@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit29` is:

**`bbc96594b4e001778eee9975372716b2`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit28.html>
