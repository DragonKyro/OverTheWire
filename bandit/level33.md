# Bandit Level 33 → 34

## Goal

You've made it to `bandit33` — congratulations! At the time of writing, this is as far as Bandit goes: **level 34 does not yet exist**. The `bandit33.html` page teases "another escape" to come, but there's no `bandit34` account to become.

This writeup is a short "what to do at the finish line" note rather than a real level solve.

## What you'll find as bandit33

### 1. Log in

```bash
$ ssh bandit33@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 32 → 33](level32.md).

### 2. Read the congratulations file

```bash
$ ls
README.txt
$ cat README.txt
```

You'll see a short message congratulating you on completing Bandit and noting that no further levels exist yet. The message typically invites you to pick another OverTheWire wargame to move on to.

## Where to go next

Now that you've finished Bandit, the natural next steps are:

- **[Leviathan](../leviathan/README.md)** — more Unix-investigation puzzles, same beginner-friendly difficulty
- **[Natas](../natas/README.md)** — server-side web security, no C required
- **[Narnia](../narnia/README.md)** — your first proper binary exploitation, with source provided

Pick whichever direction (web vs. binary vs. "more Linux puzzles") sounds most fun.

## If level 34 gets added

Watch <https://overthewire.org/wargames/bandit/bandit34.html>. The page currently says "At this moment, level 34 does not exist yet." When that changes, the puzzle will (per the `bandit33.html` tease) be another escape — likely a restricted-shell or limited-environment breakout similar in spirit to [level 32 → 33](level32.md).

## The answer

There is no password to print here — there's no `bandit34` account to log in as. You're done with Bandit.

> ⚠️ OverTheWire may introduce level 34 in the future. If `bandit34.html` no longer reads "level 34 does not exist yet," this writeup needs to be rewritten into a real walkthrough.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit33.html>
- Next-level placeholder: <https://overthewire.org/wargames/bandit/bandit34.html>
