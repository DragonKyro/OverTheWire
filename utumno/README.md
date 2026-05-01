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
| 0 → 1 | [🔒 Not yet written](level00.md) |
| 1 → 2 | [🔒 Not yet written](level01.md) |
| 2 → 3 | [🔒 Not yet written](level02.md) |
| 3 → 4 | [🔒 Not yet written](level03.md) |
| 4 → 5 | [🔒 Not yet written](level04.md) |
| 5 → 6 | [🔒 Not yet written](level05.md) |
| 6 → 7 | [🔒 Not yet written](level06.md) |
| 7 → 8 | [🔒 Not yet written](level07.md) |
| 8 → 9 | [🔒 Not yet written](level08.md) |

## Disclaimer

OverTheWire may rotate passwords. Any password printed in a per-level writeup here was correct when the level was solved — if it fails for you, re-solve using the same technique.

## References

- Official game page: <https://overthewire.org/wargames/utumno/>
