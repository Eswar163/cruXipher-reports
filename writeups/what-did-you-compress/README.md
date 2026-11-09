# what-did-you-compress



[Original report, page 21](../../cruXipher_CTF_BreakingBad_Report.pdf#page=21)

**Category** Misc / custom compression

The challenge supplied a known plaintext/compressed pair and flag.lzw. The sample bulletin describes a Fisher-Yates
symbol shuffle, a xorshift-family generator, low-bit-first packing, wider initial codes and index key 0745B6. These clues
constrain the codec enough to test against exact known plaintext.

The supplied decoder uses these parameters:

```
Parameter                                                                 Reconstructed value
```

```
Literal alphabet                                                          Shuffled byte values at codes 0-255
```

```
Reset / end                                                               256 / 257
```

```
First dynamic entry                                                       258
```

```
Initial width                                                             10 bits
```

```
Bit order                                                                 Least significant bit first
```

```
Parameter                                                                  Reconstructed value
```

```
Width growth                                                               When next index reaches 2**width
```

```
PRNG                                                                       xorshift32, shifts 13 left, 17 right, 5 left
```

```
Shuffle                                                                    Descending Fisher-Yates, swap index x % (i + 1)
```

```
Seed                                                                       0x0745B6
```

The code generates the shuffled alphabet, uses LZW's previous-entry/first-byte rule to extend the dictionary, and
compares decoded sample bytes with sample.txt. Running the archived script locally confirmed EXACT SAMPLE MATCH
True for all 2323 bytes. Decoding the supplied flag stream with the same seed returned:

```
m3ss3d_up_compXsion
```

The description explicitly requests a cruXipher{...} wrapper, giving:

```
cruXipher{m3ss3d_up_compXsion}
```

The uppercase X is present in the decoded bytes and is retained. From the working directory:

```
python what-did-you-compress/solve.py
```

The bulletin claims each tape gets a different table and says the first slots hold control codes. The working solver instead
uses literal slots 0-255 and reuses the sample seed successfully. Exact sample recovery supports the implemented
parameters; the bulletin wording is not silently substituted for the observed codec. Also, the reset branch only clears
previous-entry state, not the whole dictionary. The supplied streams decode, but this exploratory script should not be
described as a general decoder for arbitrary reset-containing streams.

Reconstructing the codec

1. Use the known plaintext as the test oracle
The goal is not merely to obtain readable output. sample.lzw must decode to every byte of sample.txt, including
whitespace and punctuation. That exact comparison distinguishes the correct parameters from a decoder that happens
to produce a plausible prefix.

2. Rebuild the shuffled literal alphabet
Start with arr = list(range(256)) and seed x with 0x0745B6. Walk the array from index 255 down to 1. Before each
swap, update xorshift32 and select j = x % (i + 1):

```
x ^= (x << 13) & 0xffffffff
x ^= x >> 17
x ^= (x << 5) & 0xffffffff
x &= 0xffffffff
j = x % (i + 1)
arr[i], arr[j] = arr[j], arr[i]
```

The masks retain the intended 32-bit state despite Python's unbounded integers. Each literal code i now represents byte
arr[i].

3. Read packed codes from the low end
The decoder interprets the byte stream as a little-endian integer and extracts a width-bit code with a right shift and
mask. Width starts at 10, not the nine bits used by many LZW examples. The first dynamic dictionary slot is 258 because
256 and 257 are control codes.

4. Extend the dictionary in the correct order
For each data code, emit its dictionary entry. If a previous entry exists, append previous + current[:1] at the next free
index. At the special next-entry case, the entry is previous + previous[:1]. Increase width when the next available
index reaches 1 << width. The exploratory script tests two growth conventions; only the matching convention
reproduces the sample exactly.

5. Apply the validated configuration to the flag stream
The complete sample match establishes the shuffle, packing and dictionary rules together. The flag's leading and trailing
codes are 256 and 257, and the same seed gives m3ss3d_up_compXsion between them. Add the event wrapper specified
in description.txt, preserving uppercase X. No extra guessed transformation is needed after decompression.

**Source files** what-did-you-compress/writeup.md, what-did-you-compress/solve.py, what-did-you-compress/description.txt, what-did-you-compress/sample.txt, what-did-you-compress/sample.lzw, what-did-you-compress/flag.lzw.
