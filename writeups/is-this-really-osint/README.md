# Is This Really OSINT?



[Original report, page 34](../../cruXipher_CTF_BreakingBad_Report.pdf#page=34)

**Category** OSINT / web / cryptography

The application at https://is-this-really-osint.cruxipher.crux-bphc.com combined six location checks, a shortlink
clue, an encrypted archive and a final cipher. Completing the six POST /api/verify checks produced an /api/status
response with all stages complete.

```
Stage                      Recorded coordinates                                         Location / clue
```

```
1                          47.4580, 19.0838                                             Budapest, OBI / B
```

```
2                          -23.887115, 32.048528                                        Massingir Dam, Landsat 3 / 3
```

```
3                          40.664111, 47.110056                                         Lake Guakhmaz, Landsat A / A
```

```
4                          36.18486, -5.43664                                           Palmones / Los Barrios, Conforama / C
```

```
5                          40.7607722, -111.8675192                                     Salt Lake City, Salon O / O
```

```
6                          41.885834, -87.624736                                        Chicago, East Lake St / Nando's / N
```

These are recorded stage answers, not newly geolocated photographs. The sequence b3acOn identifies
https://tinyurl.com/b3acOn, recorded as redirecting to
https://github.com/randombsgo234-pixel/ubiquitous-garbanzo. The completed session's /api/key returned the
archive password th3-m4p-1s-n0t-th3-t3rr1t0ry-F4wCKY. This is a puzzle key used to open the challenge artifact;
session cookies remain redacted.

The repository contained notes.7z; opening it with the returned password exposed LAST AND FINAL I PROMISE.txt. That
note contained:

```
drwL{50z3u1m35_vv3_sm4g_y45_hu3_051o7_w3_f1r_4y0og_vv3_j4z}
```

The Vigenere key BACON comes from the stage clue after removing 3. Decryption preserves case, leaves digits and
punctuation untouched, and advances the key only on letters. The result is:

```
cruX{50m3t1m35_th3_fl4g_w45_th3_051n7_w3_d1d_4l0ng_th3_w4y}
```

The short cruX prefix is supported by the notes' correct response for challenge 13 and a later already_solved response.
Decrypting and re-encrypting the preserved ciphertext confirms the exact result. The website, shortlink and repository
were not revisited; current contents are not asserted. This chain demonstrates why an application key, archive password
and final cipher key must be tracked as separate intermediate artifacts.

Following the complete clue chain

1. Finish the six location checks
Use the stage coordinates listed above with POST /api/verify. The preserved record establishes the accepted
coordinates and clue letters, but does not preserve the original photos or a complete account of locating each scene.
After the sixth stage, /api/status reported:

```
{"completedStages":6,"totalStages":6,"completed":true}
```

This completed state is needed before requesting the application key. The six clues combine as b3acOn, whose case is
retained for the shortlink.

2. Retrieve the archive key from the completed session
The original request included a session cookie. Its value is omitted here, but the endpoint and the returned
challenge-derived password are retained. This value opens the archive; treating it as a final flag skips a later required
layer.

3. Follow the shortlink to the repository

```
curl -sS -i -L https://tinyurl.com/b3acOn
```

The recorded redirect points to randombsgo234-pixel/ubiquitous-garbanzo on GitHub. Its notes.7z is the encrypted
artifact. The repository and redirect are historical evidence, not assertions about their present contents.

4. Open the archive and preserve the ciphertext
The original 7-Zip extraction used password th3-m4p-1s-n0t-th3-t3rr1t0ry-F4wCKY. The extracted LAST AND FINAL I
PROMISE.txt contains a flag-shaped ciphertext beginning drwL{. Since the prefix itself is encrypted, wrapping the
ciphertext in a new event prefix would not solve the puzzle.

5. Apply the separate Vigenere key
Remove 3 from B3ACON to obtain BACON. Subtract its repeated letter shifts only for alphabetic characters, preserving the
original case and leaving numbers, underscores and braces untouched. The first four ciphertext letters drwL become
cruX; the key position continues across the rest of the message rather than restarting inside braces.

6. Verify the plaintext and retain the short prefix
Re-encrypting the recovered plaintext with BACON reproduces the note exactly. The preserved CTFd responses are
correct followed by already_solved. Together these support the exact short-prefix flag; expanding it to cruXipher
would alter the accepted string.

**Source files** combined-writings.md.
