# Leviathan

> A small, no-programming-required wargame about poking at Unix binaries with your eyes, ears, and basic shell tools.

## About the game

Leviathan is an 8-level SSH wargame (levels 0 through 7). Each level gives you a user account whose home directory contains small hints or binaries. The job is to figure out what each binary does — usually by reading strings, tracing its behaviour, or abusing its filesystem interactions — and get it to spit out the next level's password.

**Difficulty:** 1/10 — beginner-friendly. No coding required; just Unix common sense.

## Purpose

Leviathan is often played right after Bandit. Where Bandit teaches the raw commands, Leviathan teaches the *habit* of investigating unknown binaries: peek at them, notice what they touch, build a theory, test it. It's a gentle bridge from "using the command line" toward "understanding what a program is doing behind the scenes."

## What you'll learn

- Inspecting unknown binaries with `strings`, `file`, `ltrace`, `strace`, and `hexdump`
- Reading process behaviour and noticing which files / commands a program reaches for
- Exploiting `$PATH` and relative-command lookups to hijack what a program executes
- Recognising and exploiting race-like filesystem tricks (symlinks, temp files)
- Reading short snippets of disassembly or decompilation when `strings` alone isn't enough

## How to connect

```bash
ssh leviathan0@leviathan.labs.overthewire.org -p 2223
```

- **Host:** `leviathan.labs.overthewire.org`
- **Port:** `2223`
- **Level 0 username:** `leviathan0`
- **Level 0 password:** `leviathan0`

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

## Disclaimer

OverTheWire may rotate passwords from time to time. Any password published in a per-level writeup here was correct when the level was solved — if it doesn't work, re-solve the level using the documented approach rather than assuming the writeup is stale.

## References

- Official game page: <https://overthewire.org/wargames/leviathan/>
