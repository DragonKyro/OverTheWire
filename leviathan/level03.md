# Leviathan Level 3 → 4

## Goal

Exactly the same shape as [level 1 → 2](level01.md): a setuid binary compares your input to a hardcoded password via `strcmp`, and `ltrace` hands you that password on a plate.

## Concepts you'll learn

- Recognising a repeated pattern — you've solved this before, so the solve is quick
- Reinforcing `ltrace` as a first-resort tool whenever a program asks for a password or a "magic word"

## Walkthrough

### 1. Look at the binary

```bash
$ ls -l
-r-sr-x--- 1 leviathan4 leviathan3 ... level3
```

Setuid `leviathan4`. Run it:

```bash
$ ./level3
Enter the password> hello
bzzzzzzzzap. WRONG
```

### 2. Trace it

```bash
$ ltrace ./level3
```

Enter anything at the prompt. Among the output you'll see:

```
strcmp("hello", "snlprintf")   = 19
```

The second argument is the real password: `snlprintf`.

Side note: depending on the binary version, the comparison string may be different (e.g., `snlprintf`, `sniprintf`, or something else). Trust `ltrace`'s output — not this writeup.

### 3. Enter the password for real

```bash
$ ./level3
Enter the password> snlprintf
[You've got shell]!
$ whoami
leviathan4
$ cat /etc/leviathan_pass/leviathan4
vuH0coox6m
```

### 4. Log in as leviathan4

```bash
$ ssh leviathan4@leviathan.labs.overthewire.org -p 2223
```

## The answer

The password for `leviathan4` is:

**`vuH0coox6m`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/leviathan/leviathan3.html>
