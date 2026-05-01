# Bandit Level 3 → 4

## Goal

The password for `bandit4` is hidden in a file inside the `inhere` directory. "Hidden" in Unix means the file name starts with a dot (`.`), and `ls` ignores those by default.

## Concepts you'll learn

- What a hidden file is on Unix (it's just a convention: names starting with `.`)
- How to reveal hidden files with `ls -a`
- Why dotfiles exist at all (config files, metadata, etc.)

## Walkthrough

### 1. Log in as bandit3

```bash
$ ssh bandit3@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 2 → 3](level02.md).

### 2. Check what's in the home directory

```bash
$ ls
```

Expected output:

```
inhere
```

A subdirectory. Move into it:

```bash
$ cd inhere
$ ls
```

Expected output:

```
(nothing)
```

`ls` shows nothing — but the level description promised a password file here. That's your hint that something is hidden.

### 3. Show hidden files

```bash
$ ls -a
```

Expected output (something like):

```
.   ..  .hidden
```

`-a` is the "all" flag: it shows every entry, including the ones whose name starts with `.`. You always see `.` (the current directory) and `..` (the parent directory), plus anything else that's hidden.

On recent iterations of the level, the hidden file has been named `...Hiding-From-You` instead of `.hidden`. The approach is identical — it's whatever file `ls -a` shows that wasn't shown by plain `ls`.

### 4. Read it

```bash
$ cat .hidden
```

(Or `cat ...Hiding-From-You` if that's the filename you see — note the three leading dots, so tab completion is the safe way.)

Expected output: a single line, the password for `bandit4`.

### 5. Log in as bandit4

```bash
$ ssh bandit4@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit4` is:

**`2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit3.html>
