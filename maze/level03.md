# Maze Level 3 → 4

## Goal
Pass a "magic" 32-bit value to the binary via `argv[1]`. The program compares `argv[1]` as an integer (or as 4 raw bytes) against `0x1337c0de` and only hands you a shell / password when they match.

## Concepts you'll learn
- How C code pulls an integer out of `argv` and compares it.
- **Little-endian** byte order on x86 — the least significant byte comes first in memory.
- Using `echo -e` (and command substitution) to inject raw bytes into `argv`.
- Reading a program's checks with `objdump -d` or `gdb`.

## How to log in

```
$ ssh maze3@maze.labs.overthewire.org -p 2225
```

Password is `DSEiCewQOv` (from level 2).

## Walkthrough

### 1. Find the magic value
Disassemble and look for a `cmp` against a conspicuous constant:

```
$ objdump -d /maze/maze3 | grep -i 1337
```

Expected output (roughly):

```
...  cmp    $0x1337c0de,%eax
```

The constant is `0x1337c0de`. The binary is **32-bit x86**, so the comparison is against a 4-byte integer.

### 2. Convert to little-endian bytes
On little-endian x86, the value `0x1337c0de` is laid out in memory as:

```
\xde \xc0 \x37 \x13
```

### 3. Pass the bytes as argv[1]
Use `echo -e` inside a command substitution to emit raw bytes directly into the argument list:

```
$ /maze/maze3 $(echo -e '\xde\xc0\x37\x13')
```

Expected output:

```
$
```

Then read the password:

```
$ cat /etc/maze_pass/maze4
vghylBpihH
```

If the binary reads `argv[1]` as a decimal/hex string rather than raw bytes, pass it as text instead:

```
$ /maze/maze3 0x1337c0de
```

Try both shapes if the first doesn't hit.

### 4. Log in as maze4

```
$ ssh maze4@maze.labs.overthewire.org -p 2225
```

## The answer

The password for `maze4` is:

**`vghylBpihH`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/maze/maze3.html>
