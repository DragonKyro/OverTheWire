# Bandit Level 2 → 3

## Goal

The password for `bandit3` is in a file whose name contains spaces: `spaces in this filename`. The shell normally treats spaces as *argument separators*, so you have to tell it to treat the whole name as one filename.

## Concepts you'll learn

- How the shell splits arguments on whitespace
- Two ways to pass a filename with spaces: quoting (`"name with spaces"`) and escaping (`name\ with\ spaces`)
- Tab completion — the fastest way to avoid typos with weird filenames

## Walkthrough

### 1. Log in as bandit2

```bash
$ ssh bandit2@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 1 → 2](level01.md).

### 2. See what's there

```bash
$ ls
```

Expected output:

```
spaces in this filename
```

One file, with spaces in its name. Try the naïve approach and watch it fail:

```bash
$ cat spaces in this filename
cat: spaces: No such file or directory
cat: in: No such file or directory
cat: this: No such file or directory
cat: filename: No such file or directory
```

The shell treated each whitespace-separated chunk as a separate argument, so `cat` went looking for four files called `spaces`, `in`, `this`, and `filename`.

### 3. Read the file — pick one of three ways

**Option A — double quotes.**

```bash
$ cat "spaces in this filename"
```

Everything inside `"..."` is treated as a single argument by the shell.

**Option B — backslash-escape each space.**

```bash
$ cat spaces\ in\ this\ filename
```

A backslash (`\`) in front of a space tells the shell "this space is part of the argument, not a separator."

**Option C — tab completion.**

Type `cat s` and press `Tab`. The shell completes the filename and automatically escapes the spaces for you:

```bash
$ cat spaces\ in\ this\ filename
```

Tab completion is the realistic way to do this at the keyboard — you never want to hand-type backslashes if you don't have to.

All three produce the same output: a single line with the password for `bandit3`.

### 4. Log in as bandit3

```bash
$ ssh bandit3@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit3` is:

**`MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit2.html>
