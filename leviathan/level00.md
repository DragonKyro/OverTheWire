# Leviathan Level 0 → 1

## Goal

The password for `leviathan1` is sitting in a file inside `leviathan0`'s home directory — but the file is in a **hidden directory**, and inside a longer file that needs to be searched rather than read.

## Concepts you'll learn

- Hidden directories — any file or folder whose name starts with `.` is invisible to plain `ls`
- `ls -a` to reveal them
- Using `grep` to pull just the interesting line out of a bigger file (instead of scrolling through it)

## Walkthrough

### 1. Log in as leviathan0

```bash
$ ssh leviathan0@leviathan.labs.overthewire.org -p 2223
```

The level-0 password is `leviathan0`.

### 2. Look around — with hidden files shown

```bash
$ ls -a
.  ..  .backup  .bash_logout  .bashrc  .profile
```

`.backup` is a hidden directory. Step into it:

```bash
$ cd .backup
$ ls
bookmarks.html
```

### 3. Search the bookmarks file

`bookmarks.html` is large — don't `cat` it and scroll. Ask `grep` to find the lines that mention "password" or "leviathan":

```bash
$ grep password bookmarks.html
```

Expected output: a handful of HTML lines, one of which contains:

```
<A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be the address for Leviathan 1 password:rioGegei8m" ADD_DATE="...">password to leviathan 1</A>
```

The string after `password:` — `rioGegei8m` — is what you want.

### 4. Log in as leviathan1

```bash
$ ssh leviathan1@leviathan.labs.overthewire.org -p 2223
```

## The answer

The password for `leviathan1` is:

**`rioGegei8m`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/leviathan/leviathan0.html>
