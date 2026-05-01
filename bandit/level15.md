# Bandit Level 15 → 16

## Goal

Like the previous level, you submit your current password to a local port — but this time port **30001**, and the service only speaks over **SSL/TLS**, so plain `nc` won't work.

## Concepts you'll learn

- What SSL/TLS is, at the "encrypted TCP" level of detail
- Using `openssl s_client` as an SSL-speaking netcat
- The `-quiet` and `-ign_eof` flags that make `openssl s_client` usable for simple interactions
- Using `ncat --ssl` as a shorter alternative (when available)

## Walkthrough

### 1. Log in as bandit15

```bash
$ ssh bandit15@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 14 → 15](level14.md).

### 2. Try the obvious wrong thing first (educational)

```bash
$ cat /etc/bandit_pass/bandit15 | nc localhost 30001
```

You'll hang or see garbage — the service is expecting a TLS handshake, not a raw stream.

### 3. Talk to the service with `openssl s_client`

`openssl s_client` opens an SSL/TLS connection to a host:port, then pipes stdin/stdout between you and the server (over the encrypted channel).

```bash
$ openssl s_client -connect localhost:30001 -quiet
```

Flags:

- `-connect localhost:30001` — where to connect
- `-quiet` — skip the certificate diagnostic spam and suppress the "DONE" banner on EOF

Once connected, type (or paste) your current password — the one you used to log in as `bandit15`:

```
<paste bandit15 password>
```

Press Enter. Expected response:

```
Correct!
<PASSWORD for bandit16>
```

Press **Ctrl-C** to close the SSL session.

### 4. (Alternative) `ncat --ssl`

If `ncat` is installed, it's a shorter way to do the same thing:

```bash
$ cat /etc/bandit_pass/bandit15 | ncat --ssl localhost 30001
```

### 5. Log in as bandit16

```bash
$ ssh bandit16@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit16` is:

**`kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit15.html>
