# Leviathan Level 4 → 5

## Goal

`leviathan4`'s home has a hidden binary that prints the next level's password — but it prints it as a long string of **binary digits** (zeros and ones). You convert those bits back to ASCII.

## Concepts you'll learn

- Recognising "binary" as a trivial encoding, not encryption
- Using `sed` to strip whitespace out of a stream
- One-liners for binary-to-ASCII conversion in Perl or Python

## Walkthrough

### 1. Find the binary

```bash
$ ls -la
.  ..  .bash_logout  .bashrc  .profile  .trash
```

Everything except `.trash` is standard. Look inside:

```bash
$ cd .trash
$ ls -l
-r-sr-x--- 1 leviathan5 leviathan4 ... bin
```

Setuid `leviathan5`. Run it:

```bash
$ ./bin
01010100 01101001 01110100 01101000 00110100 01100011 01101111 01101011 01100101 01101001 00001010
```

Eleven 8-bit groups, the last one being `00001010` — that's `0x0A`, a newline. The first ten groups are the 10-character password.

### 2. Strip whitespace and decode

Pipe through `sed` to remove the spaces, then through Perl to turn the 0/1 string into ASCII:

```bash
$ ./bin | sed 's/ //g' | perl -lpe '$_=pack"B*",$_'
Tith4cokei
```

What the pipeline is doing:

- `sed 's/ //g'` — replace every space with nothing (collapse to one long binary string). `g` means "every occurrence on the line."
- `perl -lpe '...'` — run the Perl expression on each line and print the result.
- `pack "B*", $_` — Perl's `pack` with the `B*` template interprets the input as a big-endian bit string and packs it into bytes. `$_` is the current line.

If you don't have Perl handy, a Python one-liner works too:

```bash
$ ./bin | tr -d ' ' | python3 -c 'import sys; s=sys.stdin.read().strip(); print("".join(chr(int(s[i:i+8],2)) for i in range(0,len(s),8)))'
Tith4cokei
```

### 3. Log in as leviathan5

```bash
$ ssh leviathan5@leviathan.labs.overthewire.org -p 2223
```

## The answer

The password for `leviathan5` is:

**`Tith4cokei`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/leviathan/leviathan4.html>
