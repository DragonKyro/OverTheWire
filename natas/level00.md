# Natas Level 0 → 1

## Goal

The `natas1` password is sitting in the HTML of the Natas 0 page, hidden in an HTML comment. Your job is to view the page source and find it.

## Concepts you'll learn

- Natas levels are HTTP-authenticated web pages at `http://natasN.natas.labs.overthewire.org/`
- "Viewing source" in a browser (Ctrl-U or right-click → View Page Source)
- That HTML comments (`<!-- ... -->`) are part of what the server sends you — they're not secret, they just don't render

## Walkthrough

### 1. Open the level in a browser

Go to:

```
http://natas0.natas.labs.overthewire.org/
```

A dialog asks for HTTP Basic credentials. Enter:

- Username: `natas0`
- Password: `natas0`

(Yes, the level-0 password is literally the username.)

### 2. The page says to look at the source

> "You can find the password for the next level on this page."

Nothing is visible on the page itself. View the source:

- Firefox / Chrome / Safari: **Ctrl-U** (or **Cmd-U** on macOS), or right-click → "View Page Source"

### 3. Find the comment

Scroll the source to near the top of the `<body>` or `<div id="content">`. You'll see something like:

```html
<!--The password for natas1 is gtVrDuiDfck831PqWsLEZy5gyDz1clto -->
```

### 4. Log in as natas1

Open `http://natas1.natas.labs.overthewire.org/` and use:

- Username: `natas1`
- Password: `gtVrDuiDfck831PqWsLEZy5gyDz1clto`

### 5. (Alternative) Do it from the command line

If you'd rather not use a browser:

```bash
$ curl -u natas0:natas0 http://natas0.natas.labs.overthewire.org/ | grep password
<!--The password for natas1 is gtVrDuiDfck831PqWsLEZy5gyDz1clto -->
```

`-u user:pass` attaches HTTP Basic credentials. `grep password` narrows the output. This is the kind of workflow you'll use for most later Natas levels.

## The answer

The password for `natas1` is:

**`gtVrDuiDfck831PqWsLEZy5gyDz1clto`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas0.natas.labs.overthewire.org/>
