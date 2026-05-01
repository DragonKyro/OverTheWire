# Leviathan Level 6 → 7

## Goal

`leviathan6`'s binary takes a **4-digit numeric code** as an argument and spawns a shell as `leviathan7` if the code is correct. There are only 10 000 possibilities, and there's no rate limiting — brute-force it.

## Concepts you'll learn

- Brute-forcing a small keyspace with a bash `for` loop
- Bash brace expansion with leading zeros (`{0000..9999}`) — same trick as [Bandit 24 → 25](../bandit/level24.md)
- Filtering the bruteforce output so you only see the one interesting result

## Walkthrough

### 1. Look at the binary

```bash
$ ls -l
-r-sr-x--- 1 leviathan7 leviathan6 ... leviathan6

$ ./leviathan6
usage: ./leviathan6 <4 digit code>

$ ./leviathan6 1234
Wrong
```

Every wrong guess gives `Wrong` and exits. The correct guess spawns a shell.

### 2. Build a working directory

```bash
$ mkdir /tmp/t6
$ cd /tmp/t6
```

### 3. Brute-force the code

```bash
$ for i in {0000..9999}; do
    ~/leviathan6 $i 2>/dev/null | grep -v Wrong && echo "code=$i" && break
  done
```

Breakdown:

- `{0000..9999}` — bash brace expansion; emits `0000`, `0001`, ..., `9999`, preserving leading zeros because the starting value has them.
- `~/leviathan6 $i` — try each code.
- `2>/dev/null` — discard stderr.
- `grep -v Wrong` — *suppress* lines containing `Wrong`. Every failed try prints `Wrong`, so this keeps us silent until the binary prints anything else.
- `&& echo "code=$i" && break` — when a try doesn't say `Wrong`, print the winning code and bail out of the loop.

Running this takes ~a minute. When it finds the match, the correct `leviathan6` actually drops you into a shell as `leviathan7` *during* the loop, so you often end up looking at a `$` prompt inside the loop's shell. If that happens:

```bash
$ whoami
leviathan7
$ cat /etc/leviathan_pass/leviathan7
ahy7MaeBo9
```

If the shell exits and the loop continues, the `echo "code=$i"` line tells you the winning code; run `~/leviathan6 <code>` manually to spawn the shell again.

### 4. Log in as leviathan7

```bash
$ ssh leviathan7@leviathan.labs.overthewire.org -p 2223
```

After logging in, `ls` shows a `CONGRATULATIONS` file:

```bash
$ cat CONGRATULATIONS
Well Done, you seem to have used a *nix system before, now try something more...interesting.
```

That's the end of Leviathan — there is no `leviathan8`. Natural next stops are [Narnia](../narnia/README.md) (your first proper binary exploitation with source code provided) or [Krypton](../krypton/README.md) (classical cryptography).

## The answer

The password for `leviathan7` is:

**`ahy7MaeBo9`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/leviathan/leviathan6.html>
