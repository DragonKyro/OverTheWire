# Bandit Level 0 → 1

## Goal

Log in to the Bandit server over SSH and find the password for the `bandit1` user. The password is sitting in a file in `bandit0`'s home directory, and once you've read it, you use it to log in as `bandit1`.

## Concepts you'll learn

- Connecting to a server with `ssh` on a non-standard port
- Listing files with `ls` and reading them with `cat`
- How OverTheWire structures each level: log in, find the password, log in as the next user

## Walkthrough

### 1. Connect to the Bandit server over SSH

Open a terminal and run:

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

What each piece means:

- `ssh` — the Secure Shell client, used to get a command line on a remote server.
- `bandit0@...` — the *username* you're logging in as (`bandit0`), then `@`, then the server's address.
- `-p 2220` — connect on **port 2220** instead of the default SSH port (22). OverTheWire runs SSH on a non-standard port so the internet's background noise doesn't pound on it.

When prompted for a password, type:

```
bandit0
```

You won't see anything as you type — that's normal for SSH passwords. Hit Enter.

On your very first connection you'll see a fingerprint warning like:

```
The authenticity of host '[bandit.labs.overthewire.org]:2220 ([...])' can't be established.
ED25519 key fingerprint is SHA256:....
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` and press Enter. After that, you'll see a short welcome banner and your shell prompt will change to something like `bandit0@bandit:~$`.

### 2. See what's in your home directory

```bash
$ ls
```

Expected output:

```
readme
```

There's a single file called `readme`. `ls` (list) just shows the names of files in the current directory — no contents, no size, just names.

### 3. Read the file

```bash
$ cat readme
```

`cat` prints the contents of a file to your terminal. Expected output is a line like:

```
Congratulations on your first session...
The password you are looking for is: <PASSWORD>
```

The part after `is:` is the password for `bandit1`. Copy it somewhere you can paste it back in a moment.

### 4. Log in as bandit1

Close your current session (type `exit` or press Ctrl-D), then:

```bash
$ ssh bandit1@bandit.labs.overthewire.org -p 2220
```

When prompted for the password, paste the one you just copied from `readme`. You're now `bandit1`, and you can move on to [level 1 → 2](level01.md).

## The answer

The password for `bandit1` is:

**`ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit0.html>
