# Bandit Level 1 → 2

## Goal

The password for `bandit2` is in a file called `-` (just a single dash) in `bandit1`'s home directory. The tricky part is that `-` has a special meaning to most command-line tools, so you have to trick them into treating it as a normal filename.

## Concepts you'll learn

- Why `-` is a magic character to most Unix tools (it usually means "stdin" or "stdout")
- Two ways to refer to a dash-named file safely: using the relative path `./-` or redirecting stdin
- The broader idea of *disambiguating* a filename from a flag

## Walkthrough

### 1. Log in as bandit1

```bash
$ ssh bandit1@bandit.labs.overthewire.org -p 2220
```

Use the password you got from [level 0 → 1](level00.md).

### 2. See what's there

```bash
$ ls
```

Expected output:

```
-
```

There's one file, and its name is literally a single dash. You'd normally read a file with `cat <filename>`, so let's try the obvious thing first:

```bash
$ cat -
```

Nothing happens. Your cursor just sits there.

What's going on? For `cat` (and many other tools), `-` is a special argument that means **"read from standard input"** — your keyboard. So `cat -` is waiting for you to type stuff. Press **Ctrl-C** to bail out.

### 3. Tell `cat` to treat `-` as a filename

There are two clean ways. Pick either one.

**Option A — use a relative path.**

```bash
$ cat ./-
```

`./` means "the current directory," so `./- ` is an unambiguous path that happens to end in `-`. `cat` no longer sees a bare dash and reads the file normally.

Expected output is a single line: the password for `bandit2`.

**Option B — use shell redirection.**

```bash
$ cat < -
```

Here you're telling the shell to *open the file named `-` and feed it to `cat`'s stdin*. `cat` never sees a `-` argument at all — it just receives input on stdin and prints it.

Same output: the password for `bandit2`.

### 4. Log in as bandit2

```bash
$ ssh bandit2@bandit.labs.overthewire.org -p 2220
```

Paste the password you just read.

## The answer

The password for `bandit2` is:

**`263JGJPfgU6LtdEvgfWU1XP5yac29mFx`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit1.html>
