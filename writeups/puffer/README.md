# puffer



[Original report, page 23](../../cruXipher_CTF_BreakingBad_Report.pdf#page=23)

**Category** Misc / hardware modeling
**Points** 400

AuthStorm advertised a device with no stored keys and a cloud service that shredded its data. robots.txt pointed to
/docs.md; the documented endpoints were /generate, /challenge and /response. The device schematic suggested a
64-stage arbiter physical unclonable function (PUF).

The 39 handout strips were character columns, each with 50000 lines. Constant columns revealed the 0x..., 0x...
format. Transposing the strips reconstructed 50000 challenge/response pairs, with omitted leading zeroes explaining
variable hexadecimal widths.

An arbiter PUF is approximately linear in cumulative-product features. The supplied solver reads challenge bits
most-significant first, adds a bias column and constructs the feature matrix:

```
s = 1.0 - 2.0 * challenge_bits.astype(np.float64)
phi = np.ones((len(s), 65), dtype=np.float64)
phi[:, 1:] = np.cumprod(s[:, ::-1], axis=1)[:, ::-1]
```

It fits 64 least-squares models against 0/1 response bits and thresholds predictions at 0.5. The writeup reports
approximately 99.1% training and 98.9% held-out per-bit accuracy; the archived script trains on all pairs and does not
preserve that held-out experiment. Residual errors are consistent with noise, but their physical cause is not established
by the supplied files.

The service's proof of work requires a decimal nonce such that SHA-256 of pow_challenge + str(nonce) starts with four
zero hex digits. After /challenge returns a new token, the model predicts the 64-bit response and posts it as a
zero-padded 0x%016x string.

The writeup records two successful responses, including challenge 0xf99f5eb3c216fcf, prediction 0xe5c26ca97b333aec,
and:

```
cruXipher{puFs_ar3_s0o0_co00o0l!}
```

The other recorded challenge/response pair is 0x161166cb7f46d5bd -> 0xf34aa401966816f0. The solver expects
strips/1.txt through strips/39.txt beside itself; those inputs are missing. Per-bit accuracy is also different from full
64-bit success probability, so the reported percentage alone is not proof of acceptance. The preserved service results are
the evidence for the successful answers.

Modeling and answering the PUF

1. Recover the API and identify the hardware model
The landing page's memory-chip clue did not imply a conventional stored password. robots.txt exposed /docs.md, and
its linked schematic showed a switched delay-line chain ending in an arbiter. A challenge controls the switching paths;
arrival order determines a response bit. This explains why the supplied challenge/response observations can train a
model.

2. Transpose the shredded observations
Read strips/1.txt through strips/39.txt as columns and join row r from each column. Constant 0, x, comma and
space columns identify the original row syntax. Parse both hexadecimal fields as 64-bit unsigned values, preserving
leading zeros through numeric conversion rather than expecting fixed-width text.

3. Generate features and fit each response bit

bits64 expands challenges most-significant bit first. Convert each bit to 1 - 2*c, calculate suffix products, and prepend
the constant bias. This creates a 50000 x 65 feature matrix. Each of the 64 response bits has its own weight vector, fitted
against its 0/1 labels. Using the wrong product direction was reported to give approximately chance-level accuracy,
supporting the chosen bit order.

4. Solve the proof of work before asking for a challenge

GET /generate supplies a token whose payload names a challenge string and difficulty. The solver tries decimal nonces
until SHA-256 of the concatenated string begins with the required zero hex digits. It sends the original token and nonce
to /challenge; decoding the new token reveals the 64-bit device challenge.

5. Predict and serialize all 64 bits
The matrix product gives one score per output bit. Scores above 0.5 become one, and the script reassembles them using
1 << (63 - b). The final request body is:

```
{"token": <challenge token>, "response": "0x%016x" % response}
```

The 0x prefix and sixteen hexadecimal positions are part of the recorded API format. The service returned Response
Verified! for both preserved examples. This is stronger evidence than per-bit accuracy alone, because the live response
must satisfy the service as a complete 64-bit value.

**Source files** puffer_writeup.md, solve_puffer.py.
