# Behemoth Level 5 → 6

## Goal

`behemoth5` doesn't read or write files — it opens a **UDP socket** and sends the next password to a local port. All you have to do is be listening on that port when it runs.

## Concepts you'll learn

- Telling apart a UDP client from a TCP client by looking at what `socket()` is called with
- Using `nc` in UDP-listen mode (`-u -l -p <port>`)
- Reading `ltrace` output for socket-related calls (`socket`, `sendto`, `bind`, `connect`)

## Walkthrough

### 1. See what the binary does

```bash
$ cd /behemoth
$ ltrace ./behemoth5
...
socket(2, 2, 0)                                         = 3
...
sendto(3, "<password>", 10, 0, ..., ...)                = 10
```

The two `2`s in `socket(2, 2, 0)` are:

- `AF_INET` = 2 (IPv4)
- `SOCK_DGRAM` = 2 (UDP)

So it's an IPv4 UDP socket. `ltrace` will also show either a `connect` or `sendto` with a `sockaddr_in` structure — and somewhere in that structure (or in a `htons(...)` call right before it) you can read the port number. Axcheron's writeup pins it at **port 1337**; if your copy of the binary uses a different port, find it with:

```bash
$ strace ./behemoth5 2>&1 | grep -E 'sendto|connect'
```

Look for `sin_port=htons(1337)` or similar. `strace` usually formats this more readably than `ltrace` does.

### 2. Start a UDP listener

In one SSH session:

```bash
$ nc -u -l -p 1337
```

- `-u` — UDP mode (default is TCP)
- `-l` — listen for incoming packets
- `-p 1337` — on this port

The listener blocks, waiting for the first packet.

### 3. Run the binary in a second session

Open a second SSH session (same user: `behemoth5`) and run:

```bash
$ /behemoth/behemoth5
```

Nothing visible happens in that session.

### 4. Back in the listener, read the password

Your `nc -u -l` now shows:

```
mayiroeche
```

That's `/etc/behemoth_pass/behemoth6`.

### 5. Why this works (a note on setuid and sockets)

`behemoth5` is setuid `behemoth6`, so while the binary runs it can read `/etc/behemoth_pass/behemoth6`. Instead of opening a file and writing to it (which would leave a trace and require permissions the sender doesn't have), it reads the password and ships it out over a UDP packet. You don't need elevated privileges to **receive** — you just need to be listening first.

## The answer

The password for `behemoth6` is:

**`mayiroeche`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/behemoth/behemoth5.html>
