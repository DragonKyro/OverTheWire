# Drifter Level 0 → 1

## Goal

Drifter 0 is a **remote syscall-proxy** challenge. There's no username/password to SSH in with — instead, a service on `drifter.labs.overthewire.org:1111` accepts **RC4-encrypted syscall requests**, executes them on the remote host, and connects back to you with results. Your job is to use that proxy to read `/flag` (or the equivalent password file) on the server.

## Concepts you'll learn

- What a **syscall proxy** is — a network service that executes Linux syscalls on behalf of a remote client
- The Linux syscall ABI at a concrete level (syscall numbers, argument registers/positions, return values)
- Using Python's `socket` and `struct` modules to speak a binary network protocol
- Writing a short RC4 implementation (or importing one) to encrypt/decrypt the protocol payloads
- "Reverse connection" design — when the *server* connects back to *you*

## Walkthrough

### 1. Read the level description and binary hints

On the OTW page you'll see a pointer to the syscall-proxy service and a reference to sources under `/drifter/` (if you have login access to a related wargame on the same host). If you don't have that context yet, you may need to use a lateral trick to pull the level's source first — that's the "meta puzzle" similar to [FormulaOne](../formulaone/README.md).

One tangible hint: reversing the proxy binary reveals an embedded password `eatmyshorts` that's used during initial handshake.

### 2. Understand the protocol

At a high level, one exchange looks like:

1. **You open a listener** on a local TCP port (ephemeral) for the remote to call back to.
2. **You connect** to `drifter.labs.overthewire.org:1111` and send your IP/port so the server knows where to reach you.
3. **The server connects back** to your listener. You now have a duplex channel.
4. **Traffic is RC4-encrypted** using a key derived from your local IP + port.
5. **You send encoded syscall requests** (syscall number + arguments); the server executes them and returns results.

### 3. Build the client

Rough sketch in Python 3:

```python
import socket, struct

def rc4(key, data):
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    i = j = 0
    out = bytearray()
    for b in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        out.append(b ^ S[(S[i] + S[j]) % 256])
    return bytes(out)

LOCAL_IP = "<your externally-routable IP>"
LOCAL_PORT = 50000

# 1. Listen for the reverse connection.
ls = socket.socket()
ls.bind(("0.0.0.0", LOCAL_PORT))
ls.listen(1)

# 2. Connect to the proxy and hand over our IP/port.
s = socket.socket()
s.connect(("drifter.labs.overthewire.org", 1111))
# (send credentials / IP info per the level's protocol — the binary tells you)

# 3. Accept the callback.
conn, addr = ls.accept()

# 4. Derive the RC4 key from LOCAL_IP + LOCAL_PORT.
KEY = (socket.inet_aton(LOCAL_IP) + struct.pack(">H", LOCAL_PORT))
```

### 4. Send the right sequence of syscalls

Goal: read `/flag` and write it back to stdout (or to our socket).

```python
# SYS_read to pull our filename into remote memory at a known address
# SYS_openat(AT_FDCWD = -100 = 0xffffff9c, "/flag", O_RDONLY)
# SYS_sendfile(stdout_fd, flag_fd, NULL, 100)
# SYS_exit
```

Each send is RC4-encrypted; each reply is RC4-decrypted. The syscall numbers you'll use (on x86-64 Linux):

- `SYS_read` = 0
- `SYS_openat` = 257
- `SYS_sendfile` = 40
- `SYS_exit` = 60

### 5. Pull the flag

If you ran it right, the sendfile's output comes back through the established channel and you see:

```
8YpAQCAuKf
```

That's the password for `drifter1`.

### 6. An honest caveat

This is the hardest wargame level documented anywhere in this repo — both because the syscall-proxy protocol is custom and because you're writing a real network client. Expect several hours with `tcpdump`/`strace` on your own machine to convince yourself the key derivation is right before the server starts responding. avishai's exploit script is worth reading for the concrete wire format.

## The answer

The password for `drifter1` is:

**`8YpAQCAuKf`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/drifter/drifter0.html>
