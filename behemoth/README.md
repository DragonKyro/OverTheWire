# Behemoth

> Classic Linux/x86 exploitation — but without the training wheels. Source code is no longer provided; you have to reverse-engineer the binary to find the bug.

## About the game

Behemoth is a 9-level SSH wargame (levels 0 through 8). Each level gives you only a compiled ELF binary, no source. Your job is to reverse it enough to understand what input it expects, find the vulnerability, exploit it, and use the resulting shell (running as the next level's user) to read `/etc/behemoth_pass/behemothN+1`.

**Difficulty:** 3/10 — the obvious next step after Narnia. Assumes you've already done stack-overflow exploitation at least once.

## Purpose

Behemoth is where you stop reading C and start reading disassembly. The bugs are the same family you saw in Narnia (overflows, format strings, race conditions, privilege issues) but you have to *find* them without a helpful `.c` file. That shift — from code review to binary review — is the core skill this game drills.

## What you'll learn

- Reverse-engineering small binaries with `objdump`, `radare2`, `ghidra`, or `gdb`
- Identifying function boundaries, stack layout, and argument-passing conventions (x86 cdecl)
- Finding vulnerabilities in assembly: unchecked `strcpy`/`gets`, format strings, integer mistakes, race conditions
- Exploiting race conditions on Unix (symlink races, TOCTOU on setuid binaries)
- Crafting shellcode and smuggling it in through environment variables or argv
- Basic return-to-libc when stack execution is blocked
- Hunting for hardcoded secrets or side-channels left by sloppy programmers

## How to connect

```bash
ssh behemoth0@behemoth.labs.overthewire.org -p 2221
```

- **Host:** `behemoth.labs.overthewire.org`
- **Port:** `2221`
- **Level 0 username:** `behemoth0`
- **Level 0 password:** `behemoth0`

## Level index

| Level | Writeup |
|------:|---------|
| 0 → 1 | ✅ [`ltrace` reveals `strcmp` password](level00.md) |
| 1 → 2 | ✅ [Stack overflow, shellcode in env var](level01.md) |
| 2 → 3 | ✅ [PATH hijack on `system("touch …")`](level02.md) |
| 3 → 4 | ✅ [Format-string GOT overwrite](level03.md) |
| 4 → 5 | ✅ [TOCTOU race via SIGSTOP + symlink](level04.md) |
| 5 → 6 | ✅ [UDP listener on port 1337](level05.md) |
| 6 → 7 | ✅ [Custom shellcode prints "HelloKitty"](level06.md) |
| 7 → 8 | ✅ [Overflow with NOP sled after return addr](level07.md) |

## Disclaimer

OverTheWire may rotate passwords without notice. Passwords printed in per-level writeups here were correct at solve time; if one fails for you, re-solve using the same method.

## References

- Official game page: <https://overthewire.org/wargames/behemoth/>
