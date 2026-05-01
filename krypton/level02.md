# Krypton Level 2 → 3

## Goal

Same cipher family as ROT13, but now the **shift is unknown**. You need to figure out how many positions the alphabet was rotated. With only 26 possible shifts, you can just try them all and eyeball which output is English.

## Concepts you'll learn

- Brute-forcing a small keyspace (26 shifts — trivial to enumerate)
- Using the level's own `encrypt2` tool for a known-plaintext check
- Writing a tiny Python or Bash loop that produces all possible decryptions

## Walkthrough

### 1. Log in and look at the files

```bash
$ ssh krypton2@krypton.labs.overthewire.org -p 2231
$ cd /krypton/krypton2
$ ls
encrypt2  krypton3  README
```

- `encrypt2` — a binary that encrypts stdin using today's key (you can use it to check guesses)
- `krypton3` — a directory owned by krypton3 containing the encrypted level-3 password

```bash
$ ls krypton3
krypton3     # the file (encrypted plaintext)
```

```bash
$ cat krypton3/krypton3
OMQEMDUEQMEK
```

Only uppercase, ends abruptly. Classical substitution cipher with a single-letter shift (Caesar).

### 2. (Optional) Use `encrypt2` to peek at the key

You can encrypt a known plaintext and compare to the output:

```bash
$ echo "A" | /krypton/krypton2/encrypt2 /tmp/out; cat /tmp/out
M
```

`A` → `M`, so the shift is **12** (A=0, M=12). But you don't even need to know that before trying brute force.

### 3. Brute-force all 26 shifts

A Python one-liner on your own machine (or from the shell prompt):

```bash
$ python3 -c "
ct = 'OMQEMDUEQMEK'
for k in range(26):
    pt = ''.join(chr((ord(c)-ord('A')-k) % 26 + ord('A')) for c in ct)
    print(f'{k:2d}: {pt}')
"
```

Expected output (partial):

```
 0: OMQEMDUEQMEK
 1: NLPDLCTDPLDJ
 2: MKOCKBSCOKCI
...
12: CAESARISEASY
...
```

Shift 12 gives you `CAESARISEASY` — the level-3 password.

### 4. (Alternative) Shell-only with `tr`

If you prefer to stay in the shell:

```bash
$ for k in $(seq 0 25); do
    printf '%2d: ' $k
    cat /krypton/krypton2/krypton3/krypton3 | tr "A-Z" "$(python3 -c "print(''.join(chr((i+$k)%26+65) for i in range(26)))")"
  done
```

Same result, same readable hit at shift 12.

### 5. Log in as krypton3

```bash
$ ssh krypton3@krypton.labs.overthewire.org -p 2231
```

## The answer

The password for `krypton3` is:

**`CAESARISEASY`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/krypton/krypton2.html>
