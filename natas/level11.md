# Natas Level 11 → 12

## Goal

The page lets you change a background colour. That preference is saved in a cookie, serialised and XOR-encrypted. Because you know one valid plaintext → ciphertext pair (the default cookie), you can **recover the XOR key** and then **forge a cookie** with `showpassword => yes`.

## Concepts you'll learn

- Why XOR with a repeating key is broken when any plaintext is known
- **Known-plaintext attacks**: `plaintext XOR ciphertext = key`
- Forging a signed/encoded cookie by re-running the same pipeline with modified data

## Walkthrough

### 1. Read the source

```php
$defaultdata = array("showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';
    for($i = 0; $i < strlen($text); $i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }
    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
        $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
        if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
            if(preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
                $mydata['bgcolor'] = $tempdata['bgcolor'];
            }
            $mydata['showpassword'] = $tempdata['showpassword'];
        }
    }
    return $mydata;
}
```

So the pipeline is:

```
server → json_encode → xor_encrypt(key) → base64_encode → cookie["data"]
```

If `showpassword == "yes"`, the server will print the password. You need to forge a cookie where `showpassword` is `"yes"`.

### 2. Recover the XOR key

You know the default plaintext (`{"showpassword":"no","bgcolor":"#ffffff"}`) and you can see the default ciphertext (look at your current cookie). Python:

```python
import base64, json
from itertools import cycle

# Grab the cookie in your browser and paste it here
cookie = "<base64 value of the 'data' cookie>"

default_plain = json.dumps({"showpassword":"no","bgcolor":"#ffffff"})
# Above may differ slightly from PHP's json_encode output; if it does, adjust.

ct = base64.b64decode(cookie)
key_stream = bytes(c ^ p for c, p in zip(ct, default_plain.encode()))
print(key_stream)
```

You'll see the key repeats — it's short. Pick the obvious repeating unit. Typically the key turns out to be `qw8J` (4 characters).

### 3. Forge the cookie

```python
import base64, json
from itertools import cycle

KEY = "qw8J"
forged_plain = json.dumps({"showpassword":"yes","bgcolor":"#ffffff"})
ct = bytes(p ^ k for p, k in zip(forged_plain.encode(), cycle(KEY.encode())))
print(base64.b64encode(ct).decode())
```

That prints your new cookie value.

### 4. Set it and reload

In dev tools → Cookies → edit the `data` cookie to your forged value. Reload the page. Now instead of "Access disallowed" you see the password.

Or in curl:

```bash
$ curl -u natas11:<pass> -b "data=<your forged base64>" http://natas11.natas.labs.overthewire.org/
```

Look for:

```
The password for natas12 is EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3
```

## The answer

The password for `natas12` is:

**`EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas11.natas.labs.overthewire.org/>
