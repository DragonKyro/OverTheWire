# CLAUDE.md — Conventions for OverTheWire Writeups

This repository contains beginner-friendly, step-by-step writeups for every level of every online wargame at [OverTheWire](https://overthewire.org/wargames/). The top-level [README.md](README.md) is the index; each game lives in its own lowercase folder with its own `README.md` and (eventually) one `levelNN.md` per level.

This file tells future Claude sessions how to write new per-level writeups consistently. Read it before producing any `levelNN.md`.

## Audience

**Absolute beginners.** Assume the reader:

- Has used a terminal a few times but is not fluent
- Has not done security challenges before
- Does not know what a SUID binary, a buffer overflow, or a base64 string is until you explain it
- Will be copy-pasting your commands and comparing output line-by-line

Do *not* write for someone who already knows the answer. Explain the *why* of each step, not just the *what*.

## File naming

- **One file per level transition:** `levelNN.md` covers the solve from level `NN` to level `NN+1`. So `level00.md` is Bandit 0 → Bandit 1.
- **Always zero-pad to two digits** (`level00.md`, `level07.md`, `level15.md`) so files sort correctly in any file browser. Never use `level0.md` or `level1.md`.
- Games with more than 100 levels don't exist in OTW today — two digits is enough.

## Structure of a per-level writeup

Every `levelNN.md` should follow this outline. Headings are Markdown `##`; keep them consistent so the whole repo feels uniform.

```markdown
# <Game> Level N → N+1

## Goal
One or two sentences describing what the player is trying to accomplish
at this level. End with "...and use that to log in as <next user>."

## Concepts you'll learn
- Bullet list of 2–5 specific tools, flags, or ideas this level drills.
- Keep each bullet short. Beginners should be able to scan this and
  decide whether they need to read background material first.

## Walkthrough

### 1. <Short name for step 1>
Explain what you're about to do and why. Then show the command:

```bash
$ some-command --flag value
```

Show the expected output in its own block:

```
expected output here
```

Then interpret the output — don't assume the reader sees what you see.
Name any new flag or concept ("The `-l` flag tells `ls` to use the long
listing format, which shows permissions and ownership.").

### 2. <Next step>
...

### 3. <Final step — usually "Log in as the next user">
```bash
$ ssh nextuserN@host -p PORT
```

## The answer

The password for `<next level>` is:

**`<password here>`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was
> correct at the time the level was solved — if it fails for you, re-solve
> the level using the method above rather than assuming the writeup is
> wrong.
```

## Style rules

- **Show commands in full.** Include the prompt character (`$`) so it's clear what's a command and what's output. Use fenced code blocks with the right language (`bash`, `text`, `c`, `python`, `http`).
- **Show expected output.** Truncate noisy output with `[...]` if it's genuinely irrelevant, but show enough to anchor the reader.
- **Explain every new flag the first time.** `grep -r`, `find -exec`, `ssh -i`, `xxd -r -p` — the reader is seeing these for the first time.
- **Name the new concept when it appears.** "This is a *setuid binary* — a program that runs with the file owner's permissions instead of the caller's."
- **Prefer short sentences.** Beginners parse short English faster than clever English.
- **Keep each step atomic.** If step 3 does three things, split it into 3a / 3b / 3c.
- **No apology language, no filler.** Don't say "as you can see" or "it's important to note." Just say the thing.
- **Use relative links.** Link back to the game README as `[game overview](README.md)` and across games as `[Bandit](../bandit/README.md)`.

## The password disclaimer (standard copy)

Every `levelNN.md` must end with the password in **bold** followed by this exact disclaimer (copy it verbatim so every writeup is consistent):

```markdown
**`<password-here>`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.
```

## Researching a level

You may look at public writeups (GitHub, personal blogs, YouTube) for a level, but:

1. **Solve it yourself first** or at least think it through before reading others. Writeups that are clearly copy-pasted are obvious and add nothing to the repo.
2. **Rewrite everything in your own words.** No paraphrased-but-recognisable sentences from another writeup.
3. **Cite the official OTW page for the level** at the bottom of your writeup if you reference it directly. Do not cite or link unofficial writeups.

## What *not* to write

- **Don't write the writeup without working through the solve.** If you couldn't actually log in as the next user using your own instructions, the writeup isn't done — no matter how clean it looks.
- **Don't invent commands.** If you're not sure a flag exists, check `man` or the tool's `--help`, don't guess.
- **Don't be clever.** A straightforward solve that the reader can follow beats a one-liner that only makes sense if you already know the answer.
- **Don't skip steps because "they're obvious."** Nothing is obvious to a beginner. Spell it out.
- **Don't update the game-level `README.md` level index speculatively.** Only flip `🔒 Not yet written` → a link once the `levelNN.md` file actually exists.

## Updating the game's level index

When you add a new `levelNN.md`, update the level-index table in the game folder's `README.md`:

- Change `| 🔒 Not yet written | (levelNN.md)` to `| ✅ [Walkthrough](levelNN.md) |` (or similar — match the existing table's style).
- Don't reorder rows; the table is in numeric order on purpose.

## When level counts turn out to be wrong

The level-index tables in each game's `README.md` were generated from what was documented on OverTheWire at the time of scaffolding. If you discover a game actually has more or fewer levels than the table shows, update the table and note the correction in that game's `README.md`. Don't silently orphan levels.

## Meta

- This repo is authored by humans and AI assistants working together. The tone should be consistent across the whole repo — don't let a new contributor's voice be noticeably different from the rest.
- When in doubt, read an existing `levelNN.md` in the same game folder and match its structure.
