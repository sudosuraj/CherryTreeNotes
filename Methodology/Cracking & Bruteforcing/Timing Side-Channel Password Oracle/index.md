# Timing Side-Channel Password Oracle

If a comparison endpoint checks a secret character-by-character with an early exit and a non-trivial per-character cost (hashing, a loop), correct characters take measurably longer to reject than wrong ones.

```bash
# for each candidate character at the current position:
#   send several requests, average response time
#   the character with the clearly higher average is probably correct
# repeat per position, extending the known prefix each time
```
Needs enough samples per candidate to beat network jitter — a handful of requests isn't enough, aim for 10+ and compare medians, not single timings.
