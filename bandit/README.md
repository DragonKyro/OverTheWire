# Bandit

> The flagship OverTheWire wargame. Aimed at absolute beginners who want to learn the Linux command line and the basic skills every other wargame assumes you already have.

## About the game

Bandit is a series of 34 levels, each hosted on a single SSH server. You log in as `banditN`, find the password for `banditN+1` somewhere in the filesystem, then log in as the next user and repeat. Every level is a self-contained puzzle that teaches one or two specific Unix tools or concepts.

**Difficulty:** Absolute beginner — no programming or security background required.

## Purpose

Bandit exists to get you comfortable in a terminal. If you've never used `ls`, `cat`, `ssh`, `grep`, `find`, or `nc` before, this is where you learn them. Later wargames (Narnia, Behemoth, Leviathan) assume you can already do everything Bandit teaches, so finishing Bandit first pays off across the whole platform.

## What you'll learn

- Logging in over SSH on a non-standard port
- Navigating the Linux filesystem (`cd`, `ls`, `pwd`, `file`)
- Reading and filtering files with `cat`, `grep`, `sort`, `uniq`, `strings`, `tr`, `base64`, `xxd`
- Finding files by name, size, permissions, or ownership using `find`
- Decoding common encodings (base64, hex, ROT13)
- Using `ssh` and `scp` with key files
- Working with network tools like `nc`, `openssl s_client`, `nmap`
- Understanding file permissions, SUID binaries, and basic privilege transitions
- Reading and editing shell scripts; following cron jobs and systemd timers
- Using `git` against a hostile/unknown repository
- Recognising common "this is not really a text file" tricks (compressed, encoded, renamed, or filesystem-quirk files)

## How to connect

All levels live on the same server. Each level has a different username; the password changes per level.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Level 0 username:** `bandit0`
- **Level 0 password:** `bandit0`

## Level index

| Level | Writeup |
|------:|---------|
| 0 → 1 | ✅ [SSH basics, reading a file](level00.md) |
| 1 → 2 | ✅ [Files named `-`](level01.md) |
| 2 → 3 | ✅ [Filenames with spaces](level02.md) |
| 3 → 4 | ✅ [Hidden (dot) files](level03.md) |
| 4 → 5 | ✅ [Text vs binary with `file`](level04.md) |
| 5 → 6 | ✅ [`find` by size and permission](level05.md) |
| 6 → 7 | ✅ [`find` by user, group, size](level06.md) |
| 7 → 8 | ✅ [`grep` for a keyword](level07.md) |
| 8 → 9 | ✅ [`sort` \| `uniq -u`](level08.md) |
| 9 → 10 | ✅ [`strings` for readable data](level09.md) |
| 10 → 11 | ✅ [`base64 -d`](level10.md) |
| 11 → 12 | ✅ [ROT13 with `tr`](level11.md) |
| 12 → 13 | ✅ [`xxd -r` + multi-layer decompress](level12.md) |
| 13 → 14 | ✅ [SSH private key login](level13.md) |
| 14 → 15 | ✅ [Netcat to a local port](level14.md) |
| 15 → 16 | ✅ [`openssl s_client` for SSL](level15.md) |
| 16 → 17 | ✅ [`nmap` scan + SSL probe](level16.md) |
| 17 → 18 | ✅ [`diff` two files](level17.md) |
| 18 → 19 | ✅ [Hostile `.bashrc` — remote command](level18.md) |
| 19 → 20 | ✅ [Setuid helper (`bandit20-do`)](level19.md) |
| 20 → 21 | ✅ [`nc -l` listener + `suconnect`](level20.md) |
| 21 → 22 | ✅ [Reading a cron script](level21.md) |
| 22 → 23 | ✅ [Cron with dynamic md5 filename](level22.md) |
| 23 → 24 | ✅ [Drop a script into a cron spool](level23.md) |
| 24 → 25 | ✅ [Brute-force a 4-digit pin](level24.md) |
| 25 → 26 | ✅ [Escape `more` via vim `:shell`](level25.md) |
| 26 → 27 | ✅ [Setuid helper again (`bandit27-do`)](level26.md) |
| 27 → 28 | ✅ [Git clone and read README](level27.md) |
| 28 → 29 | ✅ [Git history leaks a secret](level28.md) |
| 29 → 30 | ✅ [Git branches](level29.md) |
| 30 → 31 | ✅ [Git tags](level30.md) |
| 31 → 32 | ✅ [Git push with `git add -f`](level31.md) |
| 32 → 33 | ✅ [Uppercase shell — use `$0`](level32.md) |
| 33 → 34 | ✅ [Game end — level 34 not yet released](level33.md) |

## Disclaimer

OverTheWire occasionally rotates passwords. Any password published in this folder's per-level writeups reflects the value at the time the writeup was solved — if it doesn't work for you, don't assume the writeup is wrong; log in with your previous level's credentials and re-solve using the same method.

## References

- Official game page: <https://overthewire.org/wargames/bandit/>
