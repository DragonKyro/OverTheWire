# Utumno Level 8 → 9

## Goal

Transition from `utumno8` to `utumno9` — the **final** level of Utumno.

## ⚠️ This writeup is a stub

The external reference used for the rest of the Utumno writeups in this repo (axcheron) does not cover level 8. Without a trusted walk-through and without being able to solve it live, this file can't give you a step-by-step solve the way the earlier levels do.

If you reach this level and solve it, please update this file with the real walkthrough — and see [CLAUDE.md](../CLAUDE.md) for the conventions to follow.

## What's typically here

Utumno 8 is the hardest level in the wargame. Based on its positioning and the game's overall difficulty progression, expect:

- Exploitation under stronger binary mitigations than earlier levels (possibly NX, possibly partial ASLR)
- A technique from the **ROP** (Return-Oriented Programming) family — chaining small gadgets in libc or the binary itself
- Or a heap-based bug (use-after-free, heap overflow into metadata) on a glibc of the era
- Possibly a format-string info leak used to defeat ASLR before a second-stage overflow

## How to approach it

1. **Start with the source / disassembly.** Same first step as every other Utumno level: `file /utumno/utumno8`, then `gdb /utumno/utumno8`, `disassemble main`, and skim for obvious bug classes.
2. **Check mitigations:** `checksec /utumno/utumno8` (or run `readelf -l` and look for `GNU_STACK`). If NX is on, plain shellcode-in-buffer won't work — you'll need ROP.
3. **Look for an info leak first.** If the binary prints *any* pointer (via `printf`, `%p`, debug messages, or side effects), that's your ASLR defeat.
4. **Chain gadgets.** Use `ROPgadget /utumno/utumno8 | grep 'pop eax ; ret'` (and similar) to build a return-to-libc or full ROP chain.
5. **pwntools is your friend.** `from pwn import *`, `elf = ELF("/utumno/utumno8")`, `libc = ELF("/lib/i386-linux-gnu/libc.so.6")` — lets you look up symbol addresses and construct chains concisely.

## Utumno is 10 levels total

Logging in as `utumno9` is the end. Natural next steps if you complete Utumno: [Maze](../maze/README.md) (harder still — reverse engineering of unusual binaries) or [Vortex](../vortex/README.md) (end-game for OTW binary challenges).

## The answer

No snapshot password is recorded here — there's no trusted reference for this level at writing time. Capture yours by solving the level.

> ⚠️ OverTheWire may rotate passwords without notice. Even if a value were published here, it would be a snapshot — re-solve the level to get the current password.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno8.html>
