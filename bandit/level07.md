# Bandit Level 7 → 8

## Goal

The password for `bandit8` is in `data.txt`, on the same line as the word **millionth**. The file is thousands of lines long, so you need a tool that finds the right line for you.

## Concepts you'll learn

- Using `grep` to find lines containing a specific string
- The basic shape of a `grep` command: `grep <pattern> <file>`
- Why scrolling through output by hand is almost always the wrong answer

## Walkthrough

### 1. Log in as bandit7

```bash
$ ssh bandit7@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 6 → 7](level06.md).

### 2. Look at the data file

```bash
$ ls
```

Expected output:

```
data.txt
```

Check how big it is:

```bash
$ wc -l data.txt
```

Expected output (approximate):

```
98567 data.txt
```

Tens of thousands of lines. Don't try to eyeball it.

### 3. Grep for the keyword

```bash
$ grep millionth data.txt
```

`grep` prints every line that contains the pattern. Expected output:

```
millionth	<PASSWORD>
```

That's a tab character between `millionth` and the password. The password is the second column.

### 4. Log in as bandit8

```bash
$ ssh bandit8@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit8` is:

**`dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit7.html>
