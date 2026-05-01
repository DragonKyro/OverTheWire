# Bandit Level 22 → 23

## Goal

Same idea as the previous level — a cron job runs as `bandit23` and drops its password in `/tmp` — but this time the filename is **computed dynamically** inside the script, based on the username. You have to compute the same filename yourself to find the file.

## Concepts you'll learn

- Reading a shell script and simulating what it does without running it
- Using `md5sum` to compute an MD5 hash of a string
- Using `cut` to pick out a specific field of space-separated output
- Recognising "the script runs as a different user" — so `$(whoami)` inside the script is *not* what you'd get running it yourself

## Walkthrough

### 1. Log in as bandit22

```bash
$ ssh bandit22@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 21 → 22](level21.md).

### 2. Find the cron entry

```bash
$ cat /etc/cron.d/cronjob_bandit23
```

Expected output:

```
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
```

Script runs every minute as `bandit23`. Look at it:

```bash
$ cat /usr/bin/cronjob_bandit23.sh
```

Expected output:

```bash
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

Walking through it as if you were the cron system running it as `bandit23`:

- `myname=$(whoami)` → `bandit23`
- `mytarget=$(echo I am user bandit23 | md5sum | cut -d ' ' -f 1)` → the MD5 of `"I am user bandit23\n"`, with the trailing `- filename` part cut off
- Writes `/etc/bandit_pass/bandit23` to `/tmp/<that hash>`

> Critical detail: you have to **simulate `bandit23` as the username**, not your own. If you run `whoami` right now it says `bandit22`, which would give the wrong hash.

### 3. Compute the hash yourself

```bash
$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
```

Expected output (one line):

```
8ca319486bfbbc3663ea0fbe81326349
```

Breaking the pipeline down:

- `echo I am user bandit23` prints the literal string (with a newline)
- `md5sum` outputs something like `8ca319486bfbbc3663ea0fbe81326349  -` (hash, two spaces, filename)
- `cut -d ' ' -f 1` splits the line on spaces and keeps only the first field — the hash

### 4. Read the file at that path

```bash
$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
```

Expected output: `bandit23`'s password.

### 5. Log in as bandit23

```bash
$ ssh bandit23@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit23` is:

**`0Zf11ioIjMVN551jX3CmStKLYqjk54Ga`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit22.html>
