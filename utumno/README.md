# Utumno

> Harder binary exploitation — the step between "I can overflow a buffer" and "I can deal with modern mitigations."

## About the game

Utumno is a 10-level SSH wargame (levels 0 through 9). It's explicitly described by OverTheWire as "a lot harder than Leviathan and a bit harder than Behemoth." Each level hands you a binary (sometimes with source, sometimes not) and expects you to exploit it for privilege escalation to the next user, whose home directory contains the next password.

**Difficulty:** 4/10 — solidly intermediate. Previous exploitation experience is assumed.

## Purpose

Utumno's role in the progression is to push you past cookbook exploitation. Levels mix techniques (overflow + format string, or race condition + path injection), add light mitigations (some non-executable stacks, some canary-protected functions), and force you to actually plan an exploit rather than follow a template.

## What you'll learn

- Combining multiple primitives in a single exploit (e.g., format-string leak to bypass ASLR, then overflow to control flow)
- Return-to-libc and basic ROP on 32-bit Linux
- Exploiting heap-based bugs (simple use-after-free, simple overflow into adjacent chunks)
- Getting comfortable in `gdb-peda` / `pwndbg` / `gef` and using them to watch an exploit unfold
- Writing non-trivial exploit scripts with pwntools
- Reading medium-sized disassembly without getting lost
- Handling input quirks: null bytes, line buffering, pipes vs ttys

## How to connect

```bash
ssh utumno0@utumno.labs.overthewire.org -p 2227
```

- **Host:** `utumno.labs.overthewire.org`
- **Port:** `2227`
- **Level 0 username:** `utumno0`
- **Level 0 password:** `utumno0`

## Level index

| Level | Writeup |
|------:|---------|
| 0 → 1 | ✅ [`LD_PRELOAD` / hook `puts` to leak memory](level00.md) |
| 1 → 2 | ✅ [Symlink `code` → `/bin/sh`](level01.md) |
| 2 → 3 | ✅ [`execve` with argc=0 + env-var payload](level02.md) |
| 3 → 4 | ✅ [Arbitrary-write primitive via XOR'd pairs](level03.md) |
| 4 → 5 | ✅ [16-bit integer truncation → huge overflow](level04.md) |
| 5 → 6 | ✅ [Same env-var / `execve` pattern as level 2](level05.md) |
| 6 → 7 | ✅ [`strtoul("-1")` sign confusion drives arbitrary write](level06.md) |
| 7 → 8 | ✅ [Overflow across `setjmp`/`longjmp` + byte brute-force](level07.md) |
| 8 → 9 | ⚠️ [Stub only — no trusted reference available](level08.md) |

## Disclaimer

OverTheWire may rotate passwords. Any password printed in a per-level writeup here was correct when the level was solved — if it fails for you, re-solve using the same technique.

## References

- Official game page: <https://overthewire.org/wargames/utumno/>
