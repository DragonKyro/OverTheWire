# Vortex Level 10 → 11

## Goal

`vortex10` seeds `rand()` with `time()` plus a small stack-based offset. Because `time()` is almost-known (seconds-since-epoch when the process started, within a few seconds of wall-clock) and the offset range is small, you capture a handful of PRNG outputs, brute-force the seed, and predict the next output — which the binary uses as the "secret."

## Concepts you'll learn

- Why seeding `rand()` with `time(NULL)` is **catastrophically** predictable
- Recovering a random seed by trying every candidate in a small window and comparing the first few outputs
- The idea that 20 samples of a 32-bit PRNG are overwhelmingly enough to uniquely identify the seed

## Walkthrough

### 1. Understand the seed space

From reversing the binary, avishai's solve identifies:

- The seed is `time(NULL) + stack_offset` where `stack_offset` is something like `esp` at a particular moment
- The stack offset ranges roughly `-0x20` to `+0x180` (300 values)
- The time difference between your attempt and the observed outputs is bounded — say ±500 seconds

### 2. Capture 20 outputs

Connect to the service, collect the first 20 PRNG outputs it exposes (often as hex-formatted integers in its response). Save as `samples.txt`.

### 3. Brute-force the seed

```python
# level10.py
import ctypes

libc = ctypes.CDLL("libc.so.6")
observed = [int(x, 16) for x in open("samples.txt").read().split()][:20]

NOW = int(__import__("time").time())
for dt in range(-500, 501):
    for off in range(-0x20, 0x180):
        seed = NOW + dt + off
        libc.srand(seed)
        match = all(libc.rand() & 0xFFFFFFFF == observed[i] for i in range(20))
        if match:
            # Seed found — predict the next value:
            print("seed:", hex(seed))
            print("next:", hex(libc.rand() & 0xFFFFFFFF))
            raise SystemExit
```

`libc.so.6` is used so the PRNG implementation exactly matches the target's — Python's own `random` module is a different algorithm.

### 4. Submit the predicted value

Feed the predicted next PRNG output back to the service (or binary) as your answer. If the seed match was real, you're through:

```
The password for vortex11 is srx196haC
```

### 5. Log in

```bash
$ ssh vortex11@vortex.labs.overthewire.org -p 5842
```

## The answer

The password for `vortex11` is:

**`srx196haC`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/vortex/vortex10.html>
