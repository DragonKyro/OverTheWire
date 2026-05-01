# Vortex

> A long, advanced wargame that builds on everything you've learned. Expect networking, reverse engineering, cryptography, and serious exploitation.

## About the game

Vortex has 28 levels (0 through 27) and is one of the oldest and most storied OverTheWire wargames. Level 0 is unusual: there's no default username/password. Instead, you connect to a TCP service on port `5842`, it sends you four 32-bit unsigned integers in host byte order, and you have to sum them and send the result back. If you do it right, the service hands you the credentials for `vortex1`, and from there it's a normal SSH grind (SSH port `5842` for the bootstrap; subsequent levels use the usual SSH access detailed on the OTW site).

**Difficulty:** Advanced.

## Purpose

Vortex is the "show me what you've got" wargame. Every earlier game taught one set of skills; Vortex mixes them into multi-step challenges and expects you to pick the right tool without prompting. It's also a reality check for exploit reliability — many Vortex levels punish sloppy offsets or blind exploit copying.

## What you'll learn

- Writing network clients (Python, C, nc) to talk to custom TCP services
- Handling binary protocols, byte-order conversions, and framing
- Deeper ROP, return-to-libc, and stack-pivot techniques on 32- and 64-bit Linux
- Heap exploitation (use-after-free, double-free, unlinking) on older glibc allocators
- Format-string exploitation for both reads and writes
- Cryptographic attacks on weakly-used primitives
- Kernel- or driver-adjacent tricks, depending on the level
- Writing careful, reproducible exploit scripts and investing in good debugging harnesses

## How to connect

Level 0 is a TCP puzzle, not an SSH login. Connect to the puzzle port first:

```bash
nc vortex.labs.overthewire.org 5842
```

Read 4 × 4 bytes as unsigned ints (host byte order), sum them modulo 2³², send back the 4-byte sum, and the service returns the `vortex1` password. From `vortex1` onward, SSH to the host on the port listed on the OTW page for each level.

- **Host:** `vortex.labs.overthewire.org`
- **Level 0 puzzle port:** `5842` (TCP, not SSH)
- **Level 1+ SSH:** use the port documented on the OTW page for the level you're solving — Vortex's SSH ports have changed historically; verify on <https://overthewire.org/wargames/vortex/>.

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
| 15 → 16 | [🔒 Not yet written](level15.md) |
| 16 → 17 | [🔒 Not yet written](level16.md) |
| 17 → 18 | [🔒 Not yet written](level17.md) |
| 18 → 19 | [🔒 Not yet written](level18.md) |
| 19 → 20 | [🔒 Not yet written](level19.md) |
| 20 → 21 | [🔒 Not yet written](level20.md) |
| 21 → 22 | [🔒 Not yet written](level21.md) |
| 22 → 23 | [🔒 Not yet written](level22.md) |
| 23 → 24 | [🔒 Not yet written](level23.md) |
| 24 → 25 | [🔒 Not yet written](level24.md) |
| 25 → 26 | [🔒 Not yet written](level25.md) |
| 26 → 27 | [🔒 Not yet written](level26.md) |

## Disclaimer

OverTheWire may rotate passwords or adjust levels. Any passwords printed here are snapshots from solve time — if one fails, re-solve using the same method.

## References

- Official game page: <https://overthewire.org/wargames/vortex/>
