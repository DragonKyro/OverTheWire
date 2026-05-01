# Natas Level 1 → 2

## Goal

Same trick as level 0 — the password is in an HTML comment — but this time the page **disables right-click** so you can't use "View Page Source" from the context menu. You need to use the keyboard shortcut or `curl`.

## Concepts you'll learn

- Disabling right-click is client-side only — it never hides anything from you
- The **Ctrl-U / Cmd-U** keyboard shortcut for "View Page Source"
- The broader lesson: *any* HTML the server sends is visible to the client, no matter what JavaScript tries to do to hide it

## Walkthrough

### 1. Log in

Go to `http://natas1.natas.labs.overthewire.org/` with:

- Username: `natas1`
- Password: `gtVrDuiDfck831PqWsLEZy5gyDz1clto`

### 2. Notice the right-click block

Right-click anywhere on the page. Instead of the usual menu, you get a JavaScript `alert()` saying you're not allowed. Ignore it — this is a client-side trick that can't hide the HTML the server already sent.

### 3. View source anyway

Use the keyboard shortcut:

- **Ctrl-U** (Windows/Linux)
- **Cmd-U** (macOS — or **Opt-Cmd-U** in some browsers)

Or open dev tools (F12) and look at the `Elements` tab.

The same kind of comment appears:

```html
<!--The password for natas2 is ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi -->
```

### 4. (Alternative) Use curl

```bash
$ curl -u natas1:gtVrDuiDfck831PqWsLEZy5gyDz1clto http://natas1.natas.labs.overthewire.org/ | grep password
```

No browser, no right-click check, no fuss.

## The answer

The password for `natas2` is:

**`ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas1.natas.labs.overthewire.org/>
