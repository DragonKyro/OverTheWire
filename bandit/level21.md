# Bandit Level 21 → 22

## Goal

A **cron job** runs periodically as `bandit22` and writes `bandit22`'s password to a predictable file in `/tmp`. Find the cron entry, follow the script it calls, and read the file it drops.

## Concepts you'll learn

- What cron is and where system cron jobs live (`/etc/cron.d/`, `/etc/cron.hourly/`, etc.)
- Reading a cron-entry file to figure out what script runs as whom
- Tracing from a scheduled script to the file(s) it creates

## Walkthrough

### 1. Log in as bandit21

```bash
$ ssh bandit21@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 20 → 21](level20.md).

### 2. Look at the system cron jobs

```bash
$ ls /etc/cron.d/
```

Expected output (several files including one for bandit22):

```
cronjob_bandit15_root  cronjob_bandit22  cronjob_bandit23  cronjob_bandit24  cronjob_bandit25_root  ...
```

Read the one for `bandit22`:

```bash
$ cat /etc/cron.d/cronjob_bandit22
```

Expected output:

```
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

So `/usr/bin/cronjob_bandit22.sh` runs **every minute** as user `bandit22`, and its output is discarded. Let's see what it does.

### 3. Read the script

```bash
$ cat /usr/bin/cronjob_bandit22.sh
```

Expected output:

```bash
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Three things to notice:

1. It writes `bandit22`'s password to `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv`
2. It `chmod 644`s that file, making it world-readable
3. Because the script runs **as `bandit22`**, the password file is created by `bandit22` and is world-readable

### 4. Read the file

```bash
$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Expected output: `bandit22`'s password.

### 5. Log in as bandit22

```bash
$ ssh bandit22@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit22` is:

**`tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit21.html>
