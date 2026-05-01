# Leviathan Level 1 → 2

## Goal

`leviathan1`'s home directory contains a small binary called `check` that asks for a password and, if you type the right one, gives you a shell as `leviathan2`. Use library-call tracing to see exactly what it's comparing against.

## Concepts you'll learn

- `ltrace` — watches C-library calls and their arguments (first seen in [Behemoth 0](../behemoth/level00.md))
- `strings` as a quick alternative — prints every readable string embedded in a binary
- How a `setuid` binary spawns a shell as its owner when you enter the correct password

## Walkthrough

### 1. Look at the binary

```bash
$ ls -l
-r-sr-x--- 1 leviathan2 leviathan1 ... check
```

`rws...` on the owner column is the setuid bit — `check` is owned by `leviathan2`, so running it as `leviathan1` makes the process's effective UID `leviathan2`.

### 2. Run it to see what it wants

```bash
$ ./check
password: hello
Wrong password, Good Bye ...
```

It wants a password and doesn't help much beyond that.

### 3. Trace the library calls

```bash
$ ltrace ./check
```

Type any password when prompted. Among the output you'll see something like:

```
strcmp("hello", "sex")                           = -1
```

The second argument is the real password: **`sex`** (yes, really — a callback to the 1995 movie *Hackers*, where a list of the most common passwords includes `love`, `secret`, `sex`, `god`).

### 4. Enter the password for real

```bash
$ ./check
password: sex
$ whoami
leviathan2
$ cat /etc/leviathan_pass/leviathan2
ougahZi8Ta
```

### 5. (Alternative) Using `strings`

If `ltrace` isn't available or you want a second way to see the password:

```bash
$ strings ./check | head -40
```

`strings` prints every sequence of 4+ printable ASCII characters in the binary — including string constants like `"sex"` sitting alongside `"password: "` and `"Welcome..."`. It's noisier than `ltrace` but still makes short work of this level.

## The answer

The password for `leviathan2` is:

**`ougahZi8Ta`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/leviathan/leviathan1.html>
