# OverTheWire Wargame Writeups

Beginner-friendly, step-by-step writeups for every level of every online wargame at [OverTheWire](https://overthewire.org/wargames/). The goal is to make these challenges approachable for someone who is new to Linux, networking, or security — not to hand out answers, but to walk through the *thinking* that gets you from "I have no idea" to "I solved it."

## The wargames

| Game | Folder | Difficulty | Focus | Levels |
|------|--------|------------|-------|-------:|
| Bandit | [bandit/](bandit/) | Absolute beginner | Linux command line fundamentals | 0 → 34 |
| Natas | [natas/](natas/) | Beginner → intermediate | Server-side web security | 0 → 34 |
| Leviathan | [leviathan/](leviathan/) | 1/10 | Unix common sense, inspecting binaries | 0 → 7 |
| Krypton | [krypton/](krypton/) | Beginner → intermediate | Classical cryptography and cryptanalysis | 0 → 6 |
| Narnia | [narnia/](narnia/) | 2/10 | First-pass binary exploitation (with source) | 0 → 9 |
| Behemoth | [behemoth/](behemoth/) | 3/10 | Binary exploitation without source | 0 → 8 |
| Utumno | [utumno/](utumno/) | 4/10 | Harder exploitation, lightly-mitigated targets | 0 → 9 |
| Maze | [maze/](maze/) | 5/10 | Unusual reverse engineering and exploitation | 0 → 8 |
| Vortex | [vortex/](vortex/) | Advanced | Mixed networking, RE, crypto, exploitation | 0 → 27 |
| Manpage | [manpage/](manpage/) | Intermediate | C-API misconceptions, auditing | 0 → 6 |
| Drifter | [drifter/](drifter/) | Advanced | Remote syscall proxy, exploitation | 0 → ~14 |
| FormulaOne | [formulaone/](formulaone/) | Advanced | Investigative problem-solving | 0 → ~5 |

> Total level counts for Drifter and FormulaOne are approximate — the OTW front pages don't always advertise them. Each game's per-folder README notes where to confirm against the live site.

## Repository layout

Each wargame lives in its own lowercase folder. Inside each folder:

- `README.md` — overview of the game: what it teaches, how to connect, and an index of all levels
- `levelNN.md` — one file per level transition (e.g. `level00.md` is the walkthrough from level 0 to level 1). Level numbers are zero-padded so files sort correctly in any file browser.

A typical per-level writeup is structured as:

1. **Goal** — what you're trying to achieve in one sentence
2. **Concepts you'll learn** — a short list of the tools or ideas this level drills
3. **Step-by-step walkthrough** — the actual commands to run and expected output, with the reasoning at each step
4. **The answer** — the next level's password in **bold**, followed by a rotation disclaimer

If a level's writeup file is missing, it simply hasn't been written yet — the game's `README.md` index will show `🔒 Not yet written` next to that level.

## A suggested learning path

You don't have to play the games in any particular order, but this is the order that generally builds skills smoothly:

1. **Bandit** — get comfortable at the Linux command line
2. **Leviathan** or **Natas** — try either "poking at binaries" or "poking at websites" to see which direction you enjoy
3. **Narnia** — your first real binary exploitation
4. **Krypton** or **Manpage** — take a break from exploitation to learn crypto or deep-audit C code
5. **Behemoth** → **Utumno** → **Maze** — progressively harder exploitation without source
6. **Vortex**, **Drifter**, **FormulaOne** — the endgame: open-ended, advanced, less hand-holding

## A note on passwords

OverTheWire does not publish their passwords anywhere and asks that solutions not be shared publicly. Per-level writeups in this repo include the password at the bottom in **bold** so a beginner who gets completely stuck can confirm their solution worked — but:

- **Don't skip to the password.** You'll learn nothing that way.
- **Passwords may rotate.** OTW periodically regenerates them. If the password in a writeup doesn't work, the method still does — re-solve the level rather than assuming the writeup is wrong.

Each per-level writeup includes a standard disclaimer reminding readers of this.

## Contributing / updating writeups

Conventions for writing new per-level writeups live in [CLAUDE.md](CLAUDE.md) at the repo root. Follow it for style, file naming, and the password-disclaimer wording so every writeup feels consistent.

## References

- OverTheWire home page: <https://overthewire.org/wargames/>
