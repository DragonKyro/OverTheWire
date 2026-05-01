# Bandit Level 12 → 13

## Goal

The password for `bandit13` is in `data.txt`, but the file has been: hexdumped → compressed → compressed again → ... multiple times. You need to reverse all the layers to get back to plain text.

This is the longest Bandit level. Take it one layer at a time.

## Concepts you'll learn

- Reading a `xxd` hexdump and turning it back into binary with `xxd -r`
- Identifying file types with `file` so you know which decompressor to use
- The common Linux compressors: `gzip`, `bzip2`, and `tar`
- Why you work in a temp directory (your home directory is read-only for this level)

## Walkthrough

### 1. Log in as bandit12

```bash
$ ssh bandit12@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 11 → 12](level11.md).

### 2. Make a working directory

You can't modify files in your home directory as `bandit12` (you don't own it). Make a scratch directory under `/tmp`:

```bash
$ mkdir /tmp/bandit12work
$ cp data.txt /tmp/bandit12work/
$ cd /tmp/bandit12work
```

### 3. Peek at the file and reverse the hexdump

```bash
$ cat data.txt
```

Expected output (first few lines):

```
00000000: 1f8b 0808 ...
00000010: 0203 ed3e ...
...
```

That's the output format of `xxd` — address, hex bytes, sometimes ASCII on the right. Reverse it with `xxd -r`:

```bash
$ xxd -r data.txt > data
$ file data
```

Expected output:

```
data: gzip compressed data, was "data2.bin", last modified: ..., from Unix
```

### 4. Decompress, check, decompress, check — until you hit ASCII text

The rest of the level is a rinse-repeat loop: run `file`, rename to the right extension, decompress, run `file` again. The chain is typically (but not guaranteed to be, exactly): **gzip → bzip2 → gzip → tar → tar → bzip2 → gzip**.

Generic pattern: if `file X` says

- `gzip compressed data` → `mv X X.gz && gunzip X.gz`
- `bzip2 compressed data` → `mv X X.bz2 && bunzip2 X.bz2`
- `POSIX tar archive` → `mv X X.tar && tar xf X.tar` (produces a new file like `data5.bin`; point `file` at that next)

One full pass looks like:

```bash
$ mv data data.gz
$ gunzip data.gz
$ file data
data: bzip2 compressed data, block size = 900k

$ mv data data.bz2
$ bunzip2 data.bz2
$ file data
data: gzip compressed data, ...

$ mv data data.gz
$ gunzip data.gz
$ file data
data: POSIX tar archive (GNU)

$ mv data data.tar
$ tar xf data.tar
$ ls
data.tar  data5.bin
$ file data5.bin
data5.bin: POSIX tar archive (GNU)

$ tar xf data5.bin
$ ls
data.tar  data5.bin  data6.bin
$ file data6.bin
data6.bin: bzip2 compressed data, ...

$ mv data6.bin data6.bz2
$ bunzip2 data6.bz2
$ file data6
data6: POSIX tar archive (GNU)

$ tar xf data6
$ ls
... data8.bin
$ file data8.bin
data8.bin: gzip compressed data, ...

$ mv data8.bin data8.gz
$ gunzip data8.gz
$ file data8
data8: ASCII text
```

ASCII text at last. Read it:

```bash
$ cat data8
The password is FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn
```

(Your intermediate filenames may differ slightly — follow whatever `file` tells you at each step.)

### 5. Log in as bandit13

```bash
$ ssh bandit13@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit13` is:

**`FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit12.html>
