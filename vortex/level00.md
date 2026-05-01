# Vortex Level 0 → 1

## Goal

Vortex 0 is unlike any other OTW bootstrap: you connect to a TCP service on port **5842**, read four 32-bit unsigned integers it sends you, sum them up, and send the sum back as a 64-bit little-endian value. If the math is right, the service hands you the SSH credentials for `vortex1`.

## Concepts you'll learn

- Writing a small Python TCP client with the `socket` module
- The `struct` module / `pwntools`' `u32`/`p64` helpers for packing and unpacking fixed-width integers
- Why a "byte order" conversation matters: **host byte order** vs. **network byte order** (the level explicitly sends host byte order, not the network-standard big-endian)

## Walkthrough

### 1. Understand the protocol

From the official level description:

> Connect to port 5842 on vortex.labs.overthewire.org and read in 4 unsigned integers in host byte order. Add these integers together and send back the results. You will then be given a username and password for vortex1.

So: **4 × 4 bytes in** → **compute sum** → **send sum back**. The output width the server expects is 8 bytes (64-bit) so your addition doesn't overflow on the server side.

### 2. Write the client

Using `pwntools`:

```python
# level0.py
from pwn import *

s = remote("vortex.labs.overthewire.org", 5842)

total = 0
for _ in range(4):
    n = u32(s.recv(4))     # read 4 bytes, interpret as little-endian u32
    total += n

s.send(p64(total))          # pack total as little-endian u64 and send
print(s.recvall(timeout=5).decode())
```

Plain Python without `pwntools`:

```python
import socket, struct
s = socket.create_connection(("vortex.labs.overthewire.org", 5842))
data = s.recv(16)
nums = struct.unpack("<IIII", data)
total = sum(nums)
s.sendall(struct.pack("<Q", total))
print(s.recv(1024).decode())
```

### 3. Run it

```bash
$ python3 level0.py
Username: vortex1 Password: Gq#qu3bF3
```

(The exact username line format may vary; read the whole response.)

### 4. SSH in as vortex1

```bash
$ ssh vortex1@vortex.labs.overthewire.org -p 5842
```

Note: the SSH port for Vortex varies between level iterations — check the OTW page for the current value. When in doubt, try 5842 first (the bootstrap port, often reused), then fall back to whatever the level page advertises.

## The answer

The password for `vortex1` is:

**`Gq#qu3bF3`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex0.html>
