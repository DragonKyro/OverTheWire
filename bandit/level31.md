# Bandit Level 31 → 32

## Goal

Clone `bandit31`'s git repo. The README instructs you to add a file called `key.txt` containing `May I come in?`, commit it, and push to `master`. The twist: `.gitignore` excludes `*.txt`, so you have to force-add. If you push successfully, the server rejects your commit with a message that contains the password.

## Concepts you'll learn

- `.gitignore` — what it does, what it doesn't do
- `git add -f <path>` — force-add a file that `.gitignore` would normally block
- Reading the remote's response on `git push` (pre-receive hook output lands in your terminal)

## Walkthrough

### 1. Log in as bandit31

```bash
$ ssh bandit31@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 30 → 31](level30.md).

### 2. Clone the repo

```bash
$ cd /tmp
$ mkdir mygit31 && cd mygit31
$ git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo
$ cd repo
$ cat README.md
```

Expected output (approximately):

```
This time your task is to push a file to the remote repository.

Details:
  File name: key.txt
  Content: 'May I come in?'
  Branch: master
```

### 3. Check `.gitignore`

```bash
$ cat .gitignore
```

Expected output:

```
*.txt
```

So a plain `git add key.txt` will silently do nothing. You need `-f`.

### 4. Create the file and force-add

```bash
$ echo "May I come in?" > key.txt
$ git add -f key.txt
$ git commit -m "add key.txt"
```

`-f` (force) tells `git add` to add the file even though `.gitignore` excludes it.

### 5. Push

```bash
$ git push origin master
```

Supply the bandit31 password when prompted. Expected output (among normal push output):

```
remote: ### Attempting to validate files... ####
remote:
remote: .oOo.
remote: Well done! Here is the password for the next level:
remote: <PASSWORD>
remote: .oOo.
remote:
remote: error: denied by pre-receive hook
```

The server's pre-receive hook prints the password, then deliberately **rejects** the push. That's expected — the push failing is the intended design, and the password is the server's "thank you."

### 6. Log in as bandit32

```bash
$ ssh bandit32@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit32` is:

**`56a9bf19c63d650ce78e6ec0354ee45e`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit31.html>
