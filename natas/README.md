# Natas

> Server-side web security, taught by progressively leaking its own passwords.

## About the game

Natas teaches web vulnerabilities through a chain of websites, one per level. Each site is hosted at `http://natasN.natas.labs.overthewire.org` and gated by HTTP Basic Auth using the username `natasN` and the password you earned from the previous level. The password for level `N+1` is always stored on the level-`N` server under `/etc/natas_webpass/natasN+1`, and your goal is to find a web vulnerability that leaks it.

**Difficulty:** Beginner-friendly entry to web security; later levels drift into intermediate territory.

## Purpose

Natas is the easiest on-ramp OverTheWire offers for web hacking. Each level introduces one specific bug class — source-disclosure, weak auth, broken access control, injection, file inclusion, and so on — with enough context that you can learn the concept by solving one puzzle, not by reading a textbook chapter.

## What you'll learn

- How HTTP requests and responses actually look (headers, cookies, status codes)
- Using the browser dev tools and `curl` to inspect and craft traffic
- Reading PHP source and spotting insecure patterns
- Attacking: source disclosure, broken auth, predictable session IDs, command injection, SQL injection (blind and error-based), local/remote file inclusion, XXE, SSRF, insecure deserialization
- Writing short scripts (Python, `curl` loops, or your language of choice) to automate blind extraction
- Thinking in terms of *trust boundaries* — which inputs the server trusts, and why that's wrong

## How to connect

Natas is entirely web-based — no SSH. Point a browser (or `curl`) at each level's URL and supply credentials via HTTP Basic Auth.

```bash
curl -u natas0:natas0 http://natas0.natas.labs.overthewire.org/
```

- **Base URL:** `http://natasN.natas.labs.overthewire.org/` (replace `N` with the level number)
- **Level 0 username:** `natas0`
- **Level 0 password:** `natas0`

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
| 27 → 28 | [🔒 Not yet written](level27.md) |
| 28 → 29 | [🔒 Not yet written](level28.md) |
| 29 → 30 | [🔒 Not yet written](level29.md) |
| 30 → 31 | [🔒 Not yet written](level30.md) |
| 31 → 32 | [🔒 Not yet written](level31.md) |
| 32 → 33 | [🔒 Not yet written](level32.md) |
| 33 → 34 | [🔒 Not yet written](level33.md) |

> Note: OverTheWire has occasionally added or removed upper-end Natas levels. Verify the highest active level on the official page before investing time in a writeup that may not exist on the live server.

## Disclaimer

OverTheWire may rotate passwords or tweak level logic. Any password shown in a per-level writeup here was correct at solve time — if it fails for you, re-solve using the same method.

## References

- Official game page: <https://overthewire.org/wargames/natas/>
