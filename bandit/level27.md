# Bandit Level 27 → 28

## Goal

From here through level 31 the game is about **git**. Each level has its own bare git repo you clone over SSH. For level 27 the password for `bandit28` is sitting plainly in the repo's README once you've cloned it.

## Concepts you'll learn

- Cloning a git repository over SSH with a non-standard SSH port
- Using `/tmp` as scratch space for git work you can't do in your home directory
- Reading a repo (files, branches, commits, tags) — the basic vocabulary you'll need for the next four levels

## Walkthrough

### 1. Log in as bandit27

```bash
$ ssh bandit27@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 26 → 27](level26.md).

### 2. Make a scratch directory

You can't clone into your home (no write permission). Use `/tmp`:

```bash
$ mkdir /tmp/mygit27
$ cd /tmp/mygit27
```

### 3. Clone the repo

The repo lives on this same server under the `bandit27-git` account. The URL and port come from the level description:

```bash
$ git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo
```

When prompted for a password, enter your **current** `bandit27` password (the one you used to SSH in). Git over SSH uses the same auth mechanism as SSH itself.

Expected output:

```
Cloning into 'repo'...
Could not create directory '/home/bandit27/.ssh'.
The authenticity of host '[localhost]:2220 ...' can't be established.
Are you sure you want to continue connecting (yes/no)? yes
bandit27-git@localhost's password:
remote: Enumerating objects: ...
Receiving objects: ...
```

Ignore the "Could not create directory" message — it's just `ssh` complaining about not being able to write `known_hosts` into your home.

### 4. Look around

```bash
$ cd repo
$ ls
README
$ cat README
```

Expected output:

```
The password to the next level is: <PASSWORD>
```

Plain as day.

### 5. Log in as bandit28

```bash
$ ssh bandit28@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit28` is:

**`0ef186ac70e04ea33b4c1853d2526fa2`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit27.html>
