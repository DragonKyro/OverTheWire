# Manpage Level 5 → 6

## Goal

Transition from `manpage5` to `manpage6` — the **final** transition in Manpage.

## ⚠️ This writeup is a stub

The external reference used for the other Manpage levels (avishai) explicitly stops at this level with a note that the author got stuck — they recorded the `manpage5` password (which lets you log in as `manpage5`) but didn't solve `manpage5`'s challenge, so there's no trusted walkthrough for getting to `manpage6`.

If you solve this level, please update this file with the real walkthrough and follow the conventions in [CLAUDE.md](../CLAUDE.md).

## How to approach it

The game-wide pattern in Manpage is **C-programming misconceptions** — each level's binary abuses a commonly-misunderstood behaviour of a standard library or syscall, and understanding the relevant manpage is usually the key. For level 5, the method is:

1. **Read the source / disassembly.** Manpage levels are small enough that `objdump -d /manpage/manpage5` or `gdb` gives you the whole picture.
2. **Identify the C function being used suspiciously.** `strncpy`, `snprintf`, `realpath`, `scanf`, `read`, `fgets`, signal handlers, `setjmp`/`longjmp`, integer conversions — any of these have footguns documented in their manpages.
3. **Read that function's `man 3 <name>`** carefully — the "BUGS" and "NOTES" sections especially. The author of Manpage picked these levels specifically because the manpage *tells you* about the misconception.
4. **Build a tiny reproducer** on your own machine until you can trigger the bug reliably, then port the technique to the level's binary.

Likely bug classes for a "final" Manpage level:

- A subtle format-string issue or `snprintf` return-value misuse
- `readdir` race / TOCTOU
- `alloca` / variable-length array stack blow-up
- Signal-handler reentrancy — `longjmp` out of a handler that left state partially updated
- `setuid` / `seteuid` ordering bugs that leave elevated privileges around

## The answer

No snapshot password is recorded here — the reference didn't cover this level. Capture yours from your own solve.

> ⚠️ OverTheWire may rotate passwords without notice. Even if a value were published here, it would be a snapshot — re-solve the level to get the current password.

## End of Manpage

Manpage has 7 users (manpage0 through manpage6). Logging in as `manpage6` is the finish line for the wargame. Natural next stops: [Utumno](../utumno/README.md) or [Maze](../maze/README.md) for more binary exploitation, or one of the cryptography / web games in this repo if you want variety.

## Reference

- Official level page: <https://overthewire.org/wargames/manpage/manpage5.html>
