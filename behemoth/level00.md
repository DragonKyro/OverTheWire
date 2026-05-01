# Behemoth Level 0 → 1

## Goal

Find the password for `behemoth1` by reverse-engineering the `behemoth0` binary. The binary asks for a password, and somewhere inside it is comparing your input to the real one — use library-call tracing to see that comparison in action.

## Concepts you'll learn

- The structure of a Behemoth level: each level has a setuid binary in `/behemoth/` owned by the *next* user
- `ltrace` — prints every C-library call a program makes (and its arguments)
- How `strcmp` leaks its arguments to `ltrace`, turning a "password check" into a gift
- Reading the next level's password from `/etc/behemoth_pass/behemoth<N+1>` once you have a shell as that user

## Walkthrough

### 1. Log in as behemoth0

```bash
$ ssh behemoth0@behemoth.labs.overthewire.org -p 2221
```

The level-0 password is `behemoth0`.

### 2. Find the binary and check its permissions

Behemoth binaries live in `/behemoth/`:

```bash
$ cd /behemoth
$ ls -l behemoth0
-r-sr-x--- 1 behemoth1 behemoth0 ... /behemoth/behemoth0
```

Read the permissions:

- `-r-s` — owner has read and the **setuid** bit. Because the owner is `behemoth1`, running this binary gives the process the effective UID of `behemoth1`.
- `r-x` — group `behemoth0` (you) can read and execute.

If you can get this binary to spawn a shell, that shell runs as `behemoth1`, and `behemoth1` can read its own password file.

### 3. Run it straight and see what it wants

```bash
$ ./behemoth0
Password:
```

It wants a password. Type anything and press Enter:

```
hello
Authentication failure.
```

No help on its own. Try again under `ltrace`.

### 4. Use `ltrace` to watch the password check

`ltrace` intercepts every call into a shared library (glibc, mostly) and prints the arguments. When a naïve program compares passwords with `strcmp`, `ltrace` prints both sides of the comparison.

```bash
$ ltrace ./behemoth0
```

Enter any password when prompted. Among the output you'll see something like:

```
strcmp("hello", "eatmyshorts")   = -1
```

The second argument is the real password the program expected: **`eatmyshorts`**.

### 5. Run it again with the real password

```bash
$ ./behemoth0
Password: eatmyshorts
$ whoami
behemoth1
```

The binary spawned a shell, and because it was setuid, the shell runs as `behemoth1`.

### 6. Read behemoth1's password file

```bash
$ cat /etc/behemoth_pass/behemoth1
aesebootiv
```

## The answer

The password for `behemoth1` is:

**`aesebootiv`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/behemoth/behemoth0.html>
