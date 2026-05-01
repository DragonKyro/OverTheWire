# Bandit Level 13 → 14

## Goal

This level doesn't give you a password to copy — instead, `bandit13` has an **SSH private key** (`sshkey.private`) that lets you log in as `bandit14` on `localhost` without a password. Once there, you read the real password file.

## Concepts you'll learn

- SSH public-key authentication — why you can log in without typing a password when the server has your public key
- Using `ssh -i <keyfile>` to pick a specific identity file
- Reading another user's password file at `/etc/bandit_pass/banditN` once you're logged in as that user

## Walkthrough

### 1. Log in as bandit13

```bash
$ ssh bandit13@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 12 → 13](level12.md).

### 2. Find the private key

```bash
$ ls
sshkey.private
$ file sshkey.private
sshkey.private: OpenSSH private key
```

### 3. Use it to SSH in as bandit14

The trick is `-i <keyfile>`: tell `ssh` to authenticate with this specific private key. You're also connecting to `localhost` (the same machine) as user `bandit14`:

```bash
$ ssh -i sshkey.private bandit14@localhost -p 2220
```

What each piece does:

- `-i sshkey.private` — use this private key as the identity
- `bandit14@localhost` — log in as `bandit14` on this same server
- `-p 2220` — same non-standard SSH port

You may get a "host authenticity" warning — type `yes`.

If SSH complains that the key's file permissions are too open (`Permissions 0644 for 'sshkey.private' are too open`), tighten them:

```bash
$ chmod 600 sshkey.private
```

### 4. Read bandit14's password file

Once logged in as `bandit14`, the password file is at a standard location:

```bash
$ cat /etc/bandit_pass/bandit14
```

This file is only readable by the user whose password it contains — that's you now, since you're `bandit14`.

Expected output: the password.

### 5. You're already logged in as bandit14

Unlike previous levels, you don't need to `ssh` again — you're already in. You can continue from this shell to [level 14 → 15](level14.md).

## The answer

The password for `bandit14` is:

**`MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit13.html>
