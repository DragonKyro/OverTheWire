# Vortex Level 9 → 10

## Goal

There's a classic-looking binary at `/vortex/vortex9`, but the level **does not actually require exploiting it**. The password for `vortex10` was left in `/var/mail/vortex9` — a mailbox file that's world-readable (or at least readable to you as `vortex9`).

This is a reconnaissance / misdirection level. The intended bug is "don't be so focused on the binary that you miss the obvious."

## Concepts you'll learn

- That "hard" levels in a binary-exploitation wargame sometimes aren't binary-exploitation levels at all
- Using `find / -name "vortex9" 2>/dev/null` (and similar) to find all references to a term across the filesystem
- `/var/mail/<user>` as a standard Unix mailbox location worth checking

## Walkthrough

### 1. Do the obvious recon first

```bash
$ find / -name "vortex9" 2>/dev/null
/vortex/vortex9
/var/mail/vortex9
/etc/vortex_pass/vortex9
```

Three hits. `/var/mail/vortex9` is new — check it:

```bash
$ cat /var/mail/vortex9
From: admin
To: vortex9

The password for vortex10 is PWilau95f
```

### 2. That's it

No exploitation needed. Log in as `vortex10`:

```bash
$ ssh vortex10@vortex.labs.overthewire.org -p 5842
```

### 3. Lesson

It's easy to spend an hour reverse-engineering `/vortex/vortex9` before realising you never needed to touch it. When you start a level, spend a couple of minutes in `/home/$USER`, `/var/mail/$USER`, `find / -user $USER 2>/dev/null`, and `find / -name "*$USER*" 2>/dev/null` before committing to a tool.

## The answer

The password for `vortex10` is:

**`PWilau95f`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex9.html>
