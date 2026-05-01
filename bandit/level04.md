# Bandit Level 4 → 5

## Goal

The password for `bandit5` is inside `inhere/`, in the only file that's **human-readable**. All the other files in that directory are binary data. Your job is to identify which one is actual text.

## Concepts you'll learn

- The difference between text and binary files
- Using the `file` command to inspect what a file *actually* contains (not just what its name suggests)
- Running a command against every file in a directory using a shell glob (`*`)

## Walkthrough

### 1. Log in as bandit4

```bash
$ ssh bandit4@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 3 → 4](level03.md).

### 2. Look around

```bash
$ ls
```

Expected output:

```
inhere
```

Move in:

```bash
$ cd inhere
$ ls
```

Expected output:

```
-file00  -file01  -file02  -file03  -file04  -file05  -file06  -file07  -file08  -file09
```

Ten files, each starting with `-`. (Those leading dashes will matter in a moment.)

### 3. Ask each file what it is

The `file` command looks inside a file and reports its type — ASCII text, PNG image, compiled binary, and so on. It's *not* going by the filename; it's sniffing the bytes.

Run it against all the files at once with the glob `*`:

```bash
$ file ./*
```

Note the `./` in front. Without it, the glob would expand to `-file00 -file01 ...`, and `file` would see those as flags because they start with `-`. `./*` expands to `./-file00 ./-file01 ...` — same files, but the leading `./` stops them from looking like flags.

Expected output (something like):

```
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
```

`data` means "some binary blob I don't have a specific name for" — gibberish to us. The one that says **`ASCII text`** is the one we want. In this example it's `./-file07` (the exact filename may differ between solves).

### 4. Read the ASCII file

```bash
$ cat ./-file07
```

Same `./` trick as before — without it, `cat` would try to interpret `-file07` as a flag.

Expected output: one line, the password for `bandit5`.

### 5. Log in as bandit5

```bash
$ ssh bandit5@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit5` is:

**`4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit4.html>
