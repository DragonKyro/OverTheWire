# Bandit Level 14 → 15

## Goal

The password for `bandit15` is handed out by a local network service: send `bandit14`'s current password to **port 30000 on `localhost`**, and you get back `bandit15`'s password.

## Concepts you'll learn

- Using `nc` (netcat) as a simple TCP client
- Piping data into a command's stdin so it gets sent over the network
- Reading password files at `/etc/bandit_pass/banditN` (you can read your own)

## Walkthrough

### 1. Log in as bandit14

```bash
$ ssh bandit14@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 13 → 14](level13.md).

### 2. Confirm you can read your own password file

```bash
$ cat /etc/bandit_pass/bandit14
```

Expected output: your current password (the one you just logged in with). This is the string you need to send to port 30000.

### 3. Send it with `nc`

```bash
$ cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

What's happening:

- `cat /etc/bandit_pass/bandit14` prints your password to stdout
- `|` pipes that into the next command
- `nc localhost 30000` opens a TCP connection to port 30000 on this machine and sends whatever it gets on stdin

Expected output from the service:

```
Correct!
<PASSWORD for bandit15>
```

If you get `Wrong!`, check you used the right password file and that there's no stray whitespace.

### 4. Log in as bandit15

```bash
$ ssh bandit15@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit15` is:

**`8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit14.html>
