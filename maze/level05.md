# Maze Level 5 → 6

## Goal
Reverse-engineer a hardcoded authentication check in the level 5 binary: find the username/password the binary computes internally and type them in at the prompt. No memory corruption this level — just disassembly and careful string handling.

## Concepts you'll learn
- Basic static reversing with `objdump -d` / `gdb` on a **32-bit x86** binary.
- Recognizing inline string comparisons and per-byte derivations.
- Typing non-printable characters into `stdin` via `$'...'` ANSI-C quoting in bash.

## How to log in

```
$ ssh maze5@maze.labs.overthewire.org -p 2225
```

Password is `fobwgnzRy0` (from level 4).

## Walkthrough

### 1. Disassemble the check
Look for the function that reads two inputs (username, password) and compares them.

```
$ objdump -d /maze/maze5 | less
```

Find the `cmp`/`strcmp`/per-byte loop. The expected username turns out to be the literal string `username`. The expected password is derived byte-by-byte — reproduce the derivation in Python or by hand, or simply read the final bytes out of memory in `gdb` right before the compare:

```
$ gdb /maze/maze5
(gdb) break *<addr of compare>
(gdb) run
(gdb) x/16c $<reg holding expected>
```

### 2. The expected credentials
Username: `username`

Password: `<>A7?B7:` — note that this contains bytes that look like ASCII but include characters (`<`, `>`, `?`, `:`) that the shell will happily mangle if quoted wrong. If your run shows non-printable bytes, escape them with `$'\xHH'` ANSI-C quoting:

```
$ printf '%s\n%s\n' 'username' $'<>A7?B7:' | /maze/maze5
```

If your disassembly shows a different expected password, trust your disassembly, not this writeup — the derivation is fixed but snapshots drift.

### 3. Grab the flag / next password
Expected output:

```
Well done! Here's your flag: dOM2C7ZKlG
```

> ⚠️ Caveat: the source writeup this is based on conflates "flag" and "next-user password." Depending on which version of the binary is deployed, `dOM2C7ZKlG` may be a flag string rather than the login password for `maze6`. Treat it as a **snapshot**. If `ssh maze6@...` rejects it, re-run the binary on your own session and use whatever string it prints there — or check `/etc/maze_pass/maze6` if you can read it from the exploited shell.

### 4. Log in as maze6

```
$ ssh maze6@maze.labs.overthewire.org -p 2225
```

## The answer

The password for `maze6` is (snapshot — see caveat above):

**`dOM2C7ZKlG`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/maze/maze5.html>
