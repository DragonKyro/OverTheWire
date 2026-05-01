# Bandit Level 8 → 9

## Goal

The password for `bandit9` is the **only line in `data.txt` that occurs exactly once**. Every other line appears multiple times.

## Concepts you'll learn

- Pipes (`|`) — chaining commands so the output of one becomes the input of the next
- `sort` — required as a prerequisite for `uniq`
- `uniq -u` — print only lines that are *unique*, i.e. that don't have duplicates

## Walkthrough

### 1. Log in as bandit8

```bash
$ ssh bandit8@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 7 → 8](level07.md).

### 2. Peek at the data

```bash
$ wc -l data.txt
```

Expected output (approximate):

```
1001 data.txt
```

A thousand lines, nearly all duplicates.

### 3. Sort, then uniq -u

The `uniq` command removes or filters *adjacent* duplicates — so if duplicates are scattered throughout the file, you must sort first so identical lines end up next to each other.

```bash
$ sort data.txt | uniq -u
```

Breaking that down:

- `sort data.txt` — print the file's lines in alphabetical order. All the duplicates now stack together.
- `|` — the **pipe**. Takes the stdout of the command on the left and hands it to the stdin of the command on the right.
- `uniq -u` — read input line-by-line and print **only** lines that are unique (no adjacent duplicate). Without `-u`, `uniq` would collapse duplicates to a single copy; with `-u`, it drops the duplicates entirely.

Expected output: a single line, the password for `bandit9`.

### 4. Log in as bandit9

```bash
$ ssh bandit9@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit9` is:

**`4CKMh1JI91bUIZZPXDqGanal4xvAg0JM`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit8.html>
