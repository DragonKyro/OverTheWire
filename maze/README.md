# Maze

> Tricky, strange, and less-documented than the usual progression. Expect to spend real time reverse-engineering and thinking sideways.

## About the game

Maze is a 9-level SSH wargame (levels 0 through 8). OverTheWire warns that "levels are tricky and some of them strange" — expect unusual formats, non-obvious entry points, and challenges that don't map cleanly onto "overflow this and get a shell."

**Difficulty:** 5/10 — intermediate to advanced. Comfortable with exploitation, reverse engineering, and scripting is assumed.

## Purpose

Maze pushes you to combine skills from every earlier wargame and apply them creatively. Where Narnia and Behemoth drill specific bug classes, Maze asks, "given this odd thing, figure it out." It's excellent practice for the kind of open-ended reverse engineering you'd hit in a CTF or a real target.

## What you'll learn

- Reading and understanding unfamiliar binary formats or protocols
- Dynamic analysis: watching a process with `strace`, `ltrace`, and a debugger to build a model of its behaviour
- More advanced reversing with Ghidra / IDA / radare2 — recognising compilers, custom calling conventions, or obfuscation
- Scripting exploits and analysis tools in Python with pwntools
- Working through levels that may involve custom VMs, encodings, or protocol quirks
- Tolerating ambiguity and documenting partial findings so you don't lose ground between sessions

## How to connect

```bash
ssh maze0@maze.labs.overthewire.org -p 2225
```

- **Host:** `maze.labs.overthewire.org`
- **Port:** `2225`
- **Level 0 username:** `maze0`
- **Level 0 password:** `maze0`

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

## Disclaimer

OverTheWire may rotate passwords without notice. Passwords in per-level writeups here were correct at solve time — if one fails for you, re-solve rather than assuming the writeup is broken.

## References

- Official game page: <https://overthewire.org/wargames/maze/>
