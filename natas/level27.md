# Natas Level 27 → 28

## Goal

There's a `users` table in MySQL with a `username VARCHAR(64)`. The app lets you register a new account. If you supply a username *longer than 64 characters*, MySQL silently truncates it to 64. So register `admin` followed by 59 spaces + `x`: MySQL stores `admin` + 59 spaces (which == 64 chars, and compares equal to the existing `admin` row after trim). Then logging in with `admin` + empty password succeeds.

## Concepts you'll learn

- MySQL's silent-truncation behaviour for varchar fields (in some versions / modes)
- How trailing-space insensitivity (`username='admin'` == `username='admin   '`) helps the attack
- Why input lengths should be validated **before** the database enforces them

## Walkthrough

### 1. View the source

The source shows a `getUser()` function that issues a plain `SELECT` by username, and a `putUser()` that `INSERT`s into the table. The username column is varchar(64). No length check in PHP.

### 2. Register a long-usernamed "admin"

The exact form/registration endpoint varies; for axcheron's solve, a form lets you create a user with a chosen username + password. Submit:

- Username: `admin` + 59 spaces + `x` (total 65 chars; MySQL truncates to 64: `admin` + 59 spaces)
- Password: anything you'll remember, e.g. `x`

### 3. Log in as the truncated admin

Log in with:

- Username: `admin` (plus optional trailing spaces — doesn't matter)
- Password: `x`

The `SELECT * FROM users WHERE username='admin'` matches **your** padded row (trailing spaces don't count in VARCHAR comparison), and the password check against `x` succeeds (your password).

Because the app internally thinks you're "admin," it prints the admin page:

```
The password for natas28 is JWwR438wkgTsNKBbcJoowyysdM82YjeF
```

## The answer

The password for `natas28` is:

**`JWwR438wkgTsNKBbcJoowyysdM82YjeF`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas27.natas.labs.overthewire.org/>
