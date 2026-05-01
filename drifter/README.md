# Drifter

> A Vortex-adjacent wargame: more networking, more reverse engineering, and a distinctive level-0 bootstrap via a remote syscall proxy.

## About the game

Drifter is in the same tradition as Vortex — open-ended, exploitation-heavy, not always friendly. Level 0 is unusual: you don't get a username/password; instead, a remote syscall-proxy service is listening on port `1111`, and you have to use it to read the `drifter0.password` file from the server. Once you have that, SSH access for the rest of the game uses the same host on port `2230`.

**Difficulty:** Advanced. Assumes comfort with exploitation, C, and scripting.

## Purpose

Drifter rewards players who enjoy "how does this actually talk to the kernel" puzzles. Level 0 alone teaches you more about the Linux syscall ABI than most textbooks. Later levels drill exploitation primitives that don't appear in the earlier wargames — unusual protocol handling, niche corners of libc, and creative privilege transitions.

## What you'll learn

- Talking to a remote syscall proxy — i.e., issuing arbitrary Linux syscalls over the network
- The Linux syscall ABI (argument passing, return conventions, errno)
- Writing small C or Python clients that marshal syscalls onto a wire protocol
- Deeper exploitation techniques, including ROP on unusual targets and less-common primitives
- Careful reverse engineering when source is missing and the binary is doing something weird
- Staying organised in a long game where a single level can take a full session

## How to connect

Level 0 first — you won't be able to SSH in as `drifter0` until you read its password via the syscall proxy:

```bash
# Level 0 service (not SSH) — use it to read drifter0.password from the server
nc drifter.labs.overthewire.org 1111
```

Once you have the `drifter0` password, the normal SSH pattern takes over:

```bash
ssh drifter0@drifter.labs.overthewire.org -p 2230
```

- **Host:** `drifter.labs.overthewire.org`
- **SSH port:** `2230`
- **Level 0 proxy port:** `1111` (custom protocol, not SSH)
- **Level 0 username:** `drifter0` (password must be read from `drifter0.password` via the proxy)

The exact level count for Drifter isn't always advertised on the front page — verify against <https://overthewire.org/wargames/drifter/> before planning writeups for every level.

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
| 9 → 10 | [🔒 Not yet written](level09.md) |
| 10 → 11 | [🔒 Not yet written](level10.md) |
| 11 → 12 | [🔒 Not yet written](level11.md) |
| 12 → 13 | [🔒 Not yet written](level12.md) |
| 13 → 14 | [🔒 Not yet written](level13.md) |
| 14 → 15 | [🔒 Not yet written](level14.md) |

> Note: Drifter's upper-level count isn't prominently documented on the OTW front page. Extend or trim this table as you confirm the live level range.

## Disclaimer

OverTheWire may rotate passwords without notice. Passwords in per-level writeups here were correct at solve time — re-solve using the same method if one fails.

## References

- Official game page: <https://overthewire.org/wargames/drifter/>
