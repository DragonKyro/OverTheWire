# Natas Level 28 → 29

## Goal

A search field encrypts your query with **AES in ECB mode** and passes the ciphertext as a URL parameter. ECB's biggest flaw is that identical plaintext blocks become identical ciphertext blocks — so you can **rearrange ciphertext blocks**, and you can craft queries of specific lengths to align attack payloads to block boundaries. Using these tricks, you smuggle a SQL injection past the app's sanitiser.

## Concepts you'll learn

- What ECB mode is and why it leaks structure
- **Block rearrangement attacks**: pick ciphertext blocks from different queries and glue them together
- Aligning a SQL-injection payload to a 16-byte AES block boundary
- Recognising the "same plaintext → same ciphertext" tell

## Walkthrough

### 1. Read the source

- A search form POSTs to a page that:
  1. Takes your plaintext query
  2. Encrypts it with AES-128-ECB using a static key
  3. Redirects you to a GET with the ciphertext in a `query` param
  4. On GET, decrypts and uses the plaintext in a SQL query (after some sanitisation)

### 2. Demonstrate ECB's block independence

Submit a query with repeated 16-byte plaintext blocks:

```
AAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBAAAAAAAAAAAAAAAA
```

Look at the resulting ciphertext. Blocks 1 and 3 will be *byte-identical* because the plaintexts were identical. That's ECB.

### 3. Build a SQLi block

You want a plaintext chunk like:

```
' UNION SELECT password FROM users WHERE username='natas29'-- 
```

Align it to 16-byte boundaries. By submitting crafted prefixes, you can produce individual ciphertext blocks that, when reassembled, decrypt to this exact payload.

### 4. Combine blocks and send

Paste the reassembled ciphertext into the `query` parameter of a GET request. The server decrypts, runs the (now injected) SQL, and returns `natas29`'s password:

```
airooCaiseiyee8he8xongien9euhe8b
```

### 5. Honest note

This level is legitimately one of the fiddliest in Natas. If you're new to ECB block attacks, it's worth stepping away to work through a short CBC/ECB tutorial (PicoCTF's crypto track, or a CryptoPals set 2 exercise) before returning. The technique is transferable to many other contexts.

## The answer

The password for `natas29` is:

**`airooCaiseiyee8he8xongien9euhe8b`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <http://natas28.natas.labs.overthewire.org/>
