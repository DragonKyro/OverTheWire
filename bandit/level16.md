# Bandit Level 16 → 17

## Goal

Somewhere on ports **31000–32000**, a service is listening that accepts `bandit16`'s password over SSL and — unlike previous levels — hands you back an **SSH private key** for `bandit17`. You have to:

1. Port-scan the range
2. Figure out which open ports speak SSL
3. Talk to the right one and save the key
4. Use the key to SSH in as `bandit17`

## Concepts you'll learn

- Port scanning a range with `nmap`
- Distinguishing services by probing (plain TCP vs TLS vs "something else")
- Saving an inline-returned SSH key to a file with strict permissions
- Logging in with `ssh -i` again (first seen in [level 13 → 14](level13.md))

## Walkthrough

### 1. Log in as bandit16

```bash
$ ssh bandit16@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 15 → 16](level15.md).

### 2. Scan the port range

```bash
$ nmap -p 31000-32000 localhost
```

`-p 31000-32000` means scan every TCP port from 31000 through 32000. Expected output (something like):

```
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown
```

Five candidate ports. You need to find which one speaks SSL *and* accepts your password.

### 3. Probe each port to see which one speaks SSL

One by one, try each port with `openssl s_client -quiet`:

```bash
$ openssl s_client -connect localhost:31790 -quiet
```

If it speaks SSL, you'll get a prompt; if not, the handshake will fail immediately. Paste your current password and press Enter.

For **most** ports you'll either get a handshake error or a response like "Wrong!" One port will accept the password and reply with something like:

```
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAvmOk...
(many lines)
...
-----END RSA PRIVATE KEY-----
```

That's the SSH private key for `bandit17`. (In practice the right port has historically been **31790**, but verify by probing — don't assume.)

### 4. Save the key and tighten its permissions

Copy the **entire** key block (including the `BEGIN` and `END` lines) into a new file:

```bash
$ nano /tmp/bandit17.key
# paste, save, exit

$ chmod 600 /tmp/bandit17.key
```

`chmod 600` makes it readable/writable only by you. SSH refuses to use keys that are readable by other users.

### 5. Log in as bandit17 with the key

```bash
$ ssh -i /tmp/bandit17.key bandit17@localhost -p 2220
```

You're now `bandit17` — read the password file:

```bash
$ cat /etc/bandit_pass/bandit17
```

### 6. (Bonus) do it without `nano`

If you want to avoid pasting by hand, you can automate the whole thing from the level-16 session:

```bash
$ (cat /etc/bandit_pass/bandit16; sleep 1) | openssl s_client -connect localhost:31790 -quiet 2>/dev/null | sed -n '/BEGIN/,/END/p' > /tmp/bandit17.key
$ chmod 600 /tmp/bandit17.key
```

The `sed -n '/BEGIN/,/END/p'` trick prints only the lines between `BEGIN RSA PRIVATE KEY` and `END RSA PRIVATE KEY`.

## The answer

The password for `bandit17` is:

**`EReVavePLFHtFlFsjn3hyzMlvSuSAcRD`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit16.html>
