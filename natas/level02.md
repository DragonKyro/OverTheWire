# Natas Level 2 → 3

## Goal

The page tells you there's nothing on it, but the source shows an `<img>` referencing a file in a `files/` folder. Browsing that folder directly reveals a `users.txt` file with the credentials you need.

## Concepts you'll learn

- **Directory listing** — when a web server is configured (or misconfigured) to show the contents of a folder, navigating to the folder URL reveals every file in it
- A familiar recon rhythm: open page → view source → follow every path you see
- Why `users.txt` should never ship on a production site

## Walkthrough

### 1. Log in and view the source

Open `http://natas2.natas.labs.overthewire.org/` (user `natas2`, password from the previous level).

View source (Ctrl-U). Among the HTML you'll see something like:

```html
<img src="files/pixel.png">
```

A tiny image in a `files/` directory.

### 2. Navigate to the directory itself

Browse to:

```
http://natas2.natas.labs.overthewire.org/files/
```

The server has directory indexing turned on, so it lists every file in `files/`:

```
Index of /files
  ../
  pixel.png
  users.txt
```

### 3. Read users.txt

```
http://natas2.natas.labs.overthewire.org/files/users.txt
```

Contents:

```
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```

`natas3`'s password is right there.

### 4. (Alternative) All in one curl

```bash
$ curl -u natas2:<password> http://natas2.natas.labs.overthewire.org/files/users.txt
```

### 5. Log in as natas3

Go to `http://natas3.natas.labs.overthewire.org/` with the new password.

## The answer

The password for `natas3` is:

**`sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas2.natas.labs.overthewire.org/>
