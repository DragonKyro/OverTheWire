# Natas Level 8 → 9

## Goal

This level has a secret check exactly like level 6 — but instead of loading the secret from a separate file, the source **encodes** it and embeds the encoded value in the PHP. You reverse the encoding chain to recover the plaintext secret.

## Concepts you'll learn

- Encoding vs. encryption (again): a public function with no secret key is *not* security
- Reading PHP source to understand a multi-step transformation and then undoing it step-by-step
- Common PHP string functions: `bin2hex`, `strrev`, `base64_encode`

## Walkthrough

### 1. View the source

```php
$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
        print "Access granted. The password for natas9 is <censored>";
    } else {
        print "Wrong secret";
    }
}
```

The encoding chain, read from inside-out, is:

1. `base64_encode` the plaintext
2. `strrev` (reverse the string)
3. `bin2hex` (encode bytes as hex)

To decode, do the inverse in reverse order:

1. `hex2bin` the encoded string
2. Reverse the result
3. `base64_decode`

### 2. Reverse the chain

A Python one-liner:

```bash
$ python3 -c "
import base64
enc = '3d3d516343746d4d6d6c315669563362'
print(base64.b64decode(bytes.fromhex(enc)[::-1]).decode())
"
oubWYf2kBq
```

Breakdown:

- `bytes.fromhex(enc)` — undo `bin2hex`
- `[::-1]` — reverse the bytes (undo `strrev`)
- `base64.b64decode(...)` — undo `base64_encode`
- `.decode()` — pretty-print

Or stay in PHP by running `php -a` and computing:

```php
<?php
$encodedSecret = "3d3d516343746d4d6d6c315669563362";
echo base64_decode(strrev(hex2bin($encodedSecret)));
```

### 3. Submit the secret

Enter `oubWYf2kBq` in the form. Response:

```
Access granted. The password for natas9 is W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl
```

## The answer

The password for `natas9` is:

**`W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas8.natas.labs.overthewire.org/>
