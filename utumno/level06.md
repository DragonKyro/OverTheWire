# Utumno Level 6 → 7

## Goal

`utumno6` parses two numeric arguments with `strtoul`. Passing `-1` wraps around to `0xFFFFFFFF` (max-u32). That value, used as an index or offset, lets you point the write target at a location you control — like the saved return address. You then write the address of shellcode (placed in an env var) into that slot.

## Concepts you'll learn

- **Integer sign confusion** — `strtoul("-1")` = `0xFFFFFFFF`, which as a signed index is very different from what the author expected
- Combining a controlled write with env-var shellcode (seen in earlier levels)
- Reading disassembly carefully to understand *which* value at *which* offset controls *what*

## Walkthrough

### 1. Disassemble

```bash
$ gdb -q /utumno/utumno6
(gdb) disassemble main
```

You'll see `strtoul` called twice, then some arithmetic using the results as indices/offsets, culminating in a memory write. Trace each value carefully to understand what the function pointer / return address gets rewritten to and from where.

### 2. Plant shellcode

```bash
$ export EGG=$(python3 -c 'import sys; sys.stdout.buffer.write(b"\x90"*500 + b"<shellcode>")')
$ /tmp/getenv EGG /utumno/utumno6
EGG = 0xffffde...
```

Pick a return target a bit into the NOP sled.

### 3. Send `-1` and a crafted second argument

Based on axcheron's solve: `-1` as the first argument wraps to `0xFFFFFFFF`; the second argument is a pointer-like value that the code uses to dereference `EGG`'s address into the return-address slot.

```bash
$ /utumno/utumno6 -1 <computed_second_arg>
$ whoami
utumno7
$ cat /etc/utumno_pass/utumno7
totiquegae
```

The exact value for the second argument depends on your binary's instruction layout and stack positions — spend time in `gdb` tracing exactly what the write does.

### 4. An honest note

Utumno 6 is genuinely hard to describe in general terms because its "fix this two-number puzzle" shape is specific to the binary. Read the disassembly until you can explain, in one sentence, "the value of argv[1] controls X and argv[2] controls Y, and together they make the program write Z to W." Once that sentence is clear, the exploit is mechanical.

## The answer

The password for `utumno7` is:

**`totiquegae`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno6.html>
