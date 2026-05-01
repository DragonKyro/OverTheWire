# Bandit Level 20 → 21

## Goal

`bandit20`'s home directory contains a setuid binary called `suconnect` (owned by `bandit21`). When you run it with a port number, it **connects to `localhost:<port>`**, reads a line, and — if that line matches `bandit20`'s current password — writes back `bandit21`'s password. You need to stand up a listener on a local port that will hand it the right line.

## Concepts you'll learn

- Running `nc -l` as a TCP server
- The difference between a program that **listens** for connections and one that **makes** connections
- Doing two things at once in one shell with `&` (background) and `;` (sequence), or in two SSH sessions
- Feeding a file into `nc`'s stdin so it's served to the first connecting client

## Walkthrough

### 1. Log in as bandit20

```bash
$ ssh bandit20@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 19 → 20](level19.md).

### 2. See what `suconnect` expects

```bash
$ ls -l
-rwsr-x--- 1 bandit21 bandit20 ... suconnect

$ ./suconnect
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it
receives the correct password from the other side, the next password is
transmitted back.
```

Clear enough: we need to act as the **server**, have `suconnect` connect to us, and send it the `bandit20` password. It replies with the next password on the same connection.

### 3. Stand up a listener and run `suconnect`

You need two things running at the same time:

- `nc -l -p <port>` listening for an inbound connection and sending it your password
- `./suconnect <port>` connecting to that port

Two easy ways to set that up:

**Option A — two SSH sessions.**

Open a second terminal and SSH in a second time as `bandit20`.

In *session 1*:

```bash
$ nc -l -p 12345
```

The listener starts and waits. It hangs — that's fine.

In *session 2*:

```bash
$ ./suconnect 12345
```

`suconnect` connects. In session 1, the listener now has a connected client. Type (or paste) `bandit20`'s password and press Enter:

```
<bandit20 password>
```

`suconnect` in session 2 responds:

```
Read: <bandit20 password>
Password matches, sending next password
```

Session 1 prints:

```
<PASSWORD for bandit21>
```

**Option B — one shell, one pipe.**

If you want to do it in a single shell:

```bash
$ cat /etc/bandit_pass/bandit20 | nc -l -p 12345 &
$ ./suconnect 12345
```

The `&` puts `nc` in the background so you can run `suconnect` from the same terminal. The password is piped into `nc`, which serves it to the first client. After `suconnect` connects and matches, the next password appears as part of `nc`'s output.

### 4. Log in as bandit21

```bash
$ ssh bandit21@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit21` is:

**`EeoULMCra2q0dSkYj561DX7s1CpBuOBt`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit20.html>
