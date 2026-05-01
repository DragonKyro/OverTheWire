# Bandit Level 9 → 10

## Goal

The password for `bandit10` is in `data.txt`, which is mostly binary garbage. The password is one of the few human-readable strings in the file, and it's preceded by several `=` characters.

## Concepts you'll learn

- `strings` — pulls printable text sequences out of a binary file
- Piping `strings` into `grep` to narrow results
- Recognising "mostly-binary file with a little text hidden in it" as a shape

## Walkthrough

### 1. Log in as bandit9

```bash
$ ssh bandit9@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 8 → 9](level08.md).

### 2. What kind of file is this?

```bash
$ file data.txt
```

Expected output:

```
data.txt: data
```

`data` means "binary-ish; not obviously text." If you try `cat data.txt`, you'll see a wall of garbage characters and probably make your terminal unhappy. Don't bother.

### 3. Extract the readable strings

```bash
$ strings data.txt
```

`strings` scans a file and prints every run of 4 or more printable ASCII characters in a row. For a mostly-binary file, this surfaces the small "islands" of text hiding inside it. You'll see pages of mostly-nonsense strings flash by.

### 4. Narrow to lines with `=` in them

```bash
$ strings data.txt | grep =
```

The level hint says the password is preceded by **several** `=` characters. Grepping for `=` narrows the `strings` output to just those candidate lines. Expected output — a handful of lines like:

```
========== something
=========== another
========= truebwKoS0aluPOYOCfSKLjnrH6zKhW9AG
```

That last line — a run of `=` followed by the password — is your answer. Copy the text after the `=` signs.

> If too many junk lines still match, tighten the pattern: `grep '====' data.txt | strings` — or just eyeball the short list.

### 5. Log in as bandit10

```bash
$ ssh bandit10@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit10` is:

**`FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit9.html>
