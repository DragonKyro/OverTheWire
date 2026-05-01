# Bandit Level 25 → 26

## Goal

`bandit25`'s home has an SSH key for `bandit26`. But `bandit26`'s login shell isn't `bash` — it's `/usr/bin/showtext`, a tiny wrapper that pages a file with `more` and then exits. You have to **break out of `more`** into a real shell before your session dies.

## Concepts you'll learn

- What a login shell is and how changing it can lock you out of normal tools
- `more`'s ability to launch an editor (`v`) on the file it's paging
- Escaping `more` → `vi`/`vim` → a real shell with `:set shell=/bin/bash` and `:shell`
- Why a **small terminal window** is the key to making `more` pause long enough to interact

## Walkthrough

### 1. Log in as bandit25

```bash
$ ssh bandit25@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 24 → 25](level24.md).

### 2. Look at what's available

```bash
$ ls
bandit26.sshkey

$ cat /etc/passwd | grep bandit26
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
```

`bandit26`'s shell is `/usr/bin/showtext`, not a normal shell. Look inside:

```bash
$ cat /usr/bin/showtext
#!/bin/sh
export TERM=linux

exec more ~/text.txt
exit 0
```

It runs `more ~/text.txt`. When `more` exits, the script `exit 0`s — killing your session.

### 3. Shrink your terminal window *before* you log in

`more` only prompts when the file doesn't fit on one screen. If your terminal is tall enough to show the whole file at once, `more` dumps the content and exits instantly — no chance to press any keys. **Resize your terminal to a few rows tall** (3–5 lines) before the next step.

### 4. SSH in as bandit26 with the key

```bash
$ ssh -i bandit26.sshkey bandit26@localhost -p 2220
```

Because your terminal is small, `more` pauses with its `--More--` prompt at the bottom.

### 5. Drop into an editor with `v`

At the `--More--` prompt, press `v`. `more` launches `vi` (or `vim`) to edit the file. You now have a real program running as `bandit26`.

### 6. From vi, start a shell

In normal mode (if you're mid-edit, press Escape first), type:

```
:set shell=/bin/bash
```

Press Enter. Then:

```
:shell
```

Press Enter. Vim suspends and drops you into a bash shell — running as `bandit26`.

### 7. Read the password

```bash
$ cat /etc/bandit_pass/bandit26
```

Expected output: `bandit26`'s password.

> Keep this shell open — [level 26 → 27](level26.md) needs it.

## The answer

The password for `bandit26` is:

**`s0773xxkk0MXfdqOfPRVr9L3jJEoSDyT`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit25.html>
