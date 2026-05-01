# Bandit Level 29 → 30

## Goal

Clone `bandit29`'s git repo. The README on `master` says there's no live password — but there are **other branches**, and one of them has the password you want.

## Concepts you'll learn

- Git branches: `git branch`, `git branch -a`, `git checkout`
- The difference between local and remote-tracking branches (`origin/dev` vs `dev`)
- Checking every branch when a target file disappoints on the default one

## Walkthrough

### 1. Log in as bandit29

```bash
$ ssh bandit29@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 28 → 29](level28.md).

### 2. Clone the repo

```bash
$ cd /tmp
$ mkdir mygit29 && cd mygit29
$ git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repo
$ cd repo
```

### 3. Look at the README on the default branch

```bash
$ cat README.md
```

Expected output (approximately):

```
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials
- username: bandit30
- password: <no passwords in production!>
```

No password here. Check what other branches exist:

### 4. List all branches

```bash
$ git branch -a
```

Expected output:

```
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
  remotes/origin/sploits-dev
```

There's a `dev` branch on the remote.

### 5. Switch to it and read the README again

```bash
$ git checkout dev
$ cat README.md
```

Expected output:

```
## credentials
- username: bandit30
- password: <PASSWORD>
```

There it is.

### 6. Log in as bandit30

```bash
$ ssh bandit30@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit30` is:

**`5b90576bedb2cc04c86a9e924ce42faf`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit29.html>
