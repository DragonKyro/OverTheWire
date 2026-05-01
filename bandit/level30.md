# Bandit Level 30 → 31

## Goal

Clone `bandit30`'s git repo. The README is empty, the history is empty, branches are boring — but there's a **git tag** pointing at something the author tucked away.

## Concepts you'll learn

- Git tags: named pointers to specific commits, usually used for releases
- Listing tags with `git tag`
- Inspecting a tag with `git show <tagname>`

## Walkthrough

### 1. Log in as bandit30

```bash
$ ssh bandit30@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 29 → 30](level29.md).

### 2. Clone the repo

```bash
$ cd /tmp
$ mkdir mygit30 && cd mygit30
$ git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repo
$ cd repo
```

### 3. Explore the usual suspects and find them boring

```bash
$ ls
README.md
$ cat README.md
just an epmty file... muahaha
$ git log --oneline
abc1234 initial commit
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```

No hidden branches, no hidden commits. Check tags:

### 4. List and inspect tags

```bash
$ git tag
secret
```

One tag, named `secret`. See what it points at:

```bash
$ git show secret
```

Expected output:

```
<PASSWORD>
```

That's the password for `bandit31`.

> How tags work: a tag is a named reference to a specific commit (or, for "annotated" tags, to an object that points to a commit). `git show <tag>` prints the commit message and diff — or, if the tag points at a blob, the blob's contents.

### 5. Log in as bandit31

```bash
$ ssh bandit31@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit31` is:

**`47e603bb428404d265f59c42920d81e5`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit30.html>
