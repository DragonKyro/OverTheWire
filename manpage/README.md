# Manpage

> A tiny wargame with a big point: the manuals are lying, or at least misleading, and real programs bleed because of it.

## About the game

Manpage is a 7-level SSH wargame (levels 0 through 6). Each level demonstrates a common misconception about a standard C library or system call — the kind of thing a careful reading of `man 3 strcpy`, `man 2 open`, or `man 7 signal` would reveal but that programmers routinely get wrong. You're expected to audit a small program, connect its behaviour back to the exact line of the manpage that warned you, and exploit the gap.

**Difficulty:** Intermediate. Some C and exploitation background helps a lot.

## Purpose

Manpage's purpose is to train your code-review eye. By the end you'll have internalised a handful of "wait, that's not actually what this API does" patterns that show up constantly in real codebases — and you'll have learned to treat `man` pages as adversarial reading material, not just reference text.

## What you'll learn

- Reading manpages for landmines: return-value quirks, buffer-size rules, TOCTOU windows, signed/unsigned surprises
- Common C pitfalls: `strncpy` not null-terminating, `realpath` with `NULL`, integer conversion rules, `sprintf` vs `snprintf`, signal-handler reentrancy
- How subtle spec-vs-implementation mismatches turn into exploitable bugs
- Short, surgical exploitation: usually one or two primitives per level, focused on the specific pitfall
- Building an auditor's checklist you can carry into real code

## How to connect

```bash
ssh manpage0@manpage.labs.overthewire.org -p 2224
```

- **Host:** `manpage.labs.overthewire.org`
- **Port:** `2224`
- **Level 0 username:** `manpage0`
- **Level 0 password:** `manpage0`

## Level index

| Level | Writeup |
|------:|---------|
| 0 → 1 | [🔒 Not yet written](level00.md) |
| 1 → 2 | [🔒 Not yet written](level01.md) |
| 2 → 3 | [🔒 Not yet written](level02.md) |
| 3 → 4 | [🔒 Not yet written](level03.md) |
| 4 → 5 | [🔒 Not yet written](level04.md) |
| 5 → 6 | [🔒 Not yet written](level05.md) |

## Disclaimer

OverTheWire may rotate passwords. Any password published in a per-level writeup here was correct at solve time — if it doesn't work, re-solve the level rather than assuming the writeup is outdated.

## References

- Official game page: <https://overthewire.org/wargames/manpage/>
