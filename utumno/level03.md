# Utumno Level 3 → 4

## Goal

`utumno3` reads 8 **pairs** of (offset, byte) — so you have 8 arbitrary byte-writes relative to a stack frame. Each "byte" is XOR'd with a small value depending on the loop iteration, so you have to pre-compute the input byte so that after the XOR the stored value is what you want. Use those 8 writes to overwrite the saved return address with a pointer to shellcode you've placed in an environment variable.

## Concepts you'll learn

- **Arbitrary-write primitives** — a bug class where user input controls both the where (offset) and the what (byte) of a write
- Reasoning about per-iteration transformations (the XOR) so your input yields the correct output byte
- Using an env var as a known-address pointer target for the overwritten return address

## Walkthrough

### 1. Read the disassembly

In `gdb`:

```bash
$ gdb -q /utumno/utumno3
(gdb) disassemble main
```

You'll see an 8-iteration loop that reads two bytes per iteration, XORs the first byte with some per-iteration constant (axcheron identifies the XOR mask as `0`, `3`, `6`, `9`, …), and writes the result to `[ebp + offset - 0x24]`.

### 2. Plan your 8 writes

You need to write 4 bytes of a return address into the saved-return-address slot. The offset from `ebp` to saved EIP is typically `+0x04` — so `offset - 0x24 == 0x04` means `offset == 0x28`. The 4 bytes of the return address go at offsets `0x28`, `0x29`, `0x2a`, `0x2b`.

But since input bytes are XOR'd, to *write* byte `B` in iteration `i`, you have to input `B ^ mask[i]`.

That only uses 4 of your 8 writes — use the other 4 to do something harmless (write back the original bytes, or target an unused stack slot).

### 3. Plant shellcode in an env var and get its address

```bash
$ export EGG=$(python3 -c 'import sys; sys.stdout.buffer.write(b"\x90"*500 + b"<shellcode>")')
$ /tmp/getenv EGG /utumno/utumno3
EGG = 0xffffde...
```

(See [Behemoth 1](../behemoth/level01.md) for the `getenv` helper.)

### 4. Send the 8 pairs

```bash
$ (echo 40 <XOR-adjusted low byte>; \
   echo 41 <XOR-adjusted next byte>; \
   echo 42 <XOR-adjusted next byte>; \
   echo 43 <XOR-adjusted high byte>; \
   echo 0 0; echo 0 0; echo 0 0; echo 0 0) | /utumno/utumno3
```

`40`, `41`, `42`, `43` in decimal are `0x28..0x2b` — the four bytes of the saved return address.

After the loop completes and the function returns, EIP lands in your NOP sled and runs the shellcode:

```bash
$ whoami
utumno4
$ cat /etc/utumno_pass/utumno4
oogieleoga
```

### 5. An honest note

This level is fiddly because (a) you have to read the disassembly carefully to find the exact XOR masks and offset formula, and (b) stack addresses drift between runs. Use `gdb` to re-verify the formula each time you rebuild, and consider wrapping the whole thing in a Python `pwntools` exploit that reads the binary's output, computes offsets on the fly, and retries on failure.

## The answer

The password for `utumno4` is:

**`oogieleoga`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno3.html>
