# FormulaOne

> A "rescued" wargame with a meta twist: finding the starting point is part of the puzzle.

## About the game

FormulaOne is an older, rescued OverTheWire wargame. It's unusual in two ways. First, the OTW page won't hand you an obvious level-0 username and password; instead, the source code for the first level lives inside *another* wargame hosted on the same server, and you have to figure out which one and go get it. Second, each level tends to demand more general problem-solving and investigation than a single exploit technique.

**Difficulty:** Advanced — intentionally under-documented.

## Purpose

FormulaOne's purpose is less "learn technique X" and more "prove you can operate without a guide." It's closer to a CTF puzzle series than a tutorial wargame: the bootstrap alone forces you to think about how the OTW servers are laid out and to poke at adjacent games.

## What you'll learn

- Reconnaissance on a multi-tenant Linux host — what files you can read, what other services exist, what lives in `/home/*` or `/srv/*`
- Piecing together context from multiple wargames hosted on the same box
- Reading unfamiliar source code quickly to decide whether it's worth exploiting
- Writing exploits and tooling without step-by-step guidance
- General problem-solving and investigative patience

## How to connect

OverTheWire lists the host and SSH port for FormulaOne, but no default level-0 password — that's by design.

```bash
# You'll know the right username/password once you've located the level-0 source
ssh formulaone0@formulaone.labs.overthewire.org -p 2232
```

- **Host:** `formulaone.labs.overthewire.org`
- **Port:** `2232`
- **Level 0 credentials:** not handed out — discover them by finding the first level's source code inside a neighbouring wargame on the same server, as hinted on the OTW page.

The exact number of levels isn't prominently advertised; verify on <https://overthewire.org/wargames/formulaone/> before committing to writeups for every level.

## Level index

| Level | Writeup |
|------:|---------|
| 0 → 1 | [🔒 Not yet written](level00.md) |
| 1 → 2 | [🔒 Not yet written](level01.md) |
| 2 → 3 | [🔒 Not yet written](level02.md) |
| 3 → 4 | [🔒 Not yet written](level03.md) |
| 4 → 5 | [🔒 Not yet written](level04.md) |
| 5 → 6 | [🔒 Not yet written](level05.md) |

> Note: This index is a best guess at the level range. Confirm against the live OTW page and adjust the table as you go.

## Disclaimer

OverTheWire may rotate passwords or re-host FormulaOne without notice. Any password published in a per-level writeup here was correct at solve time; re-solve if the password fails rather than assuming the writeup is broken.

## References

- Official game page: <https://overthewire.org/wargames/formulaone/>
