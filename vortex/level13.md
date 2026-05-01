# Vortex Level 13 → 14

## Goal

`vortex13` behaves differently when launched with **zero argv entries** (`argc == 0`) — a check that can't be triggered from a normal shell, because shells always pass at least argv[0] (the program name). You need a small C harness that `execve`s the binary with an empty argv.

## ⚠️ This writeup is a stub

The reference I used for earlier Vortex levels (avishai) explicitly marks this level as unsolved ("stuck!" / "BAD :(" on the source page). Without a trusted walkthrough, the following covers the setup and general technique only — the final exploit was not successfully documented.

## Concepts you'll learn

- That many Unix utilities assume `argc ≥ 1` (argv[0] is the program name), so "what if argc is 0" is an edge case with exploitable consequences
- Using `execve(path, (char*[]){NULL}, env)` in C to pass a truly-empty argv
- How to check the binary's `argc` branch in `gdb` by `break` + `info registers` on the `cmp ... , 0x0` that gates the two code paths

## Walkthrough

### 1. Confirm the `argc == 0` branch

```bash
$ gdb -q /vortex/vortex13
(gdb) disassemble main
```

Look for something like:

```
cmp    DWORD PTR [ebp+0x8], 0x0
je     <vulnerable_path>
```

The `[ebp+0x8]` value is `argc`. Taking the `je` leads you somewhere interesting — presumably the unsafe code path you need to reach.

### 2. Launcher program

```c
// /tmp/launch13.c
#include <unistd.h>
int main(void) {
    char *argv[] = { NULL };
    char *env[]  = { NULL };
    execve("/vortex/vortex13", argv, env);
    return 0;
}
```

```bash
$ gcc -m32 /tmp/launch13.c -o /tmp/launch13
$ /tmp/launch13
```

`execve` with a truly-zero argv calls `vortex13` with `argc == 0` — reachable the `je` branch.

### 3. What to do from there

The part that the reference got stuck on: **what the `argc == 0` branch actually does and how to turn that into code execution**. Options worth investigating from your own reverse engineering:

- Does the branch read uninitialised stack memory? If so, you may need to control `env` (or argv layout before the execve) to pre-populate usable bytes.
- Does the branch `exit()` immediately and skip privilege drops? If so, the vulnerability might be "the binary with `argc == 0` exits before dropping privileges, and *another* copy of state stays alive" — worth looking for side-effects.
- Does it dispatch to an alternate code path that does unsafe `strcpy`/`printf` on argv[something] where "something" is out of bounds? The `execve` in the launcher lets you put carefully chosen content at `environ[0]` / `environ[1]` since those addresses are now at the top of the stack.

## The answer

No password is published here. The reference source didn't complete this level, and I didn't fabricate a solve. Capture `vortex14`'s password by completing the exploit on a live session.

> ⚠️ OverTheWire may rotate passwords without notice. Even if a value were published here, it would be a snapshot — re-solve the level to get the current password.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex13.html>
