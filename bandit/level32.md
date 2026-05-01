# Bandit Level 32 → 33

## Goal

`bandit32` is dropped into a weird shell that **uppercases every command you type** — so `ls` becomes `LS`, `cat` becomes `CAT`, and nothing works. You need to break out of this "uppercase shell" and get a normal bash.

The trick is a beautiful little shell quirk: the variable `$0` holds the name of the currently running shell, and the shell doesn't uppercase variable expansions.

## Concepts you'll learn

- Restricted / custom shells — what they restrict and what they don't
- The shell variable `$0` (the name/path of the current shell)
- Why expanding `$0` as a command drops you into a regular shell

## Walkthrough

### 1. Log in as bandit32

```bash
$ ssh bandit32@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 31 → 32](level31.md).

You'll see a banner like:

```
WELCOME TO THE UPPERCASE SHELL
>> 
```

### 2. Watch it butcher your commands

```
>> ls
sh: 1: LS: not found
>> cat /etc/passwd
sh: 1: CAT: not found
```

Every command you type is uppercased before the shell tries to run it. Anything case-sensitive (every real Linux command) breaks.

### 3. Execute `$0`

At the `>>` prompt, type:

```
>> $0
```

Press Enter.

What happens:

- The shell's input-mangler doesn't touch the `$` — it uppercases `$0` to `$0` (still `$0`, since there's no case to change)
- Then the shell *expands* `$0`, which is the shell's name/path — something like `/bin/sh` or `bash`
- Running that command spawns a **new, normal shell** with no uppercase filter

You're now in a regular shell:

```
$ whoami
bandit33
```

### 4. Read the password

```bash
$ cat /etc/bandit_pass/bandit33
```

Expected output: the password for `bandit33`.

### 5. Log in as bandit33

```bash
$ ssh bandit33@bandit.labs.overthewire.org -p 2220
```

Use the password you just recovered.

> Other tricks that also work here, for the curious: `/bin/bash` (the shell converts the input but the slashes and lowercase are preserved by expansion tricks), or using environment-variable substitution cleverly. `$0` is the simplest and most famous.

## The answer

The password for `bandit33` is:

**`c9c3199ddf4121b10cf581a98d51caee`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit32.html>
