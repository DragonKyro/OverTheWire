# Narnia

> Your first proper dive into binary exploitation — with source code included, so you can see the bug before you exploit it.

## About the game

Narnia is a 10-level SSH wargame (levels 0 through 9) where each level hands you both a vulnerable binary (running as a higher-privileged user) and its C source. You read the code, spot the memory bug, craft the input that triggers it, and walk out with the next level's password (usually stored in `/etc/narnia_pass/narniaN+1` and readable only by the binary's effective user).

**Difficulty:** 2/10 — gentle but real. This is where many players first exploit a stack buffer overflow.

## Purpose

Narnia's purpose is to make the concept of a *vulnerability* concrete. Every level is a small program with one obvious-in-hindsight mistake, and your job is to weaponise that mistake. Because you always have the source, you're never guessing — you're learning the pattern: overflow → overwrite → control execution.

## What you'll learn

- Reading small C programs and spotting unsafe patterns (`gets`, unchecked `strcpy`, format-string calls with attacker-controlled format, integer overflows, off-by-ones)
- Running binaries under `gdb`, setting breakpoints, reading the stack
- Using `objdump`, `nm`, and `checksec` to survey a binary
- Crafting exploit inputs by hand and with Python/pwntools
- Understanding what a stack frame looks like and why overwriting a return address gives you code execution
- Writing and placing shellcode (when NX isn't in the way) or working around NX with return-to-libc / ROP on harder levels
- Environment-variable tricks for leaking or aligning addresses

## How to connect

```bash
ssh narnia0@narnia.labs.overthewire.org -p 2226
```

- **Host:** `narnia.labs.overthewire.org`
- **Port:** `2226`
- **Level 0 username:** `narnia0`
- **Level 0 password:** `narnia0`

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

OverTheWire may rotate passwords without notice. Per-level writeups record the password at solve time — if yours doesn't match, re-solve with the same method rather than assuming the writeup is stale.

## References

- Official game page: <https://overthewire.org/wargames/narnia/>
