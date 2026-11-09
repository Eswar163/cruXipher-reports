# santas-nice-list



[Original report, page 33](../../cruXipher_CTF_BreakingBad_Report.pdf#page=33)

**Category** OSINT
**Points** 500

The challenge supplies 41 addresses and emphasizes delivering gifts at an exact midnight time. This suggests examining
the relationship between location, longitude and time. Several listed businesses lie close to particular longitude
meridians:

```
Location                   Approximate longitude                                          Mean-solar time offset
```

```
Old Arabi Food Store       -90 degrees                                                    -6 hours
```

```
ASR Food Market,           -75 degrees                                                    -5 hours
Philadelphia
```

```
Kaka Kariyana, Sirsa       +75 degrees                                                    +5 hours
```

```
AK & Co., Anchorage        -150 degrees                                                   -10 hours
```

```
Ernie's Coffee Shop        -120 degrees                                                   -8 hours
```

```
Takano cafe, Shiraz        +52.5 degrees                                                  +3 hours 30 minutes
```

```
Roadhaven Cafe,            +150 degrees                                                   +10 hours
Marulan
```

Earth rotates through 360 degrees in 24 hours, giving 15 degrees per hour. An east-positive longitude therefore gives an
approximate mean-solar time offset:

```
offset = longitude / 15
UTC time of mean-solar midnight = (-offset) mod 24
```

These offsets differ from official civil time zones. Sirsa, for example, is near 75 degrees east, corresponding to
approximately UTC+5 in mean-solar time, while Indian Standard Time is UTC+5:30. The longitude observations support
the geographic interpretation, but do not establish the location-to-character extraction.

Recognizing the encoded sequence
The intermediate sequence associated with the 41-location list is:

```
MNZHKWDJOBUGK4T3KMYFEUZGRXDA4TUNBJXINDSPU
```

Its uppercase letters and digits 2 through 7 match the standard Base32 alphabet. The original sequence has 41
characters; an unpadded standard Base32 message cannot have a length congruent to 1 modulo 8. A character is
therefore missing if this is intended to be a complete Base32 message.

Inserting S at one-based position 23 produces a 42-character sequence:

```
MNZHKWDJOBUGK4T3KMYFEUSZGRXDA4TUNBJXINDSPU
```

The insertion reconstructs a missing character. Its justification from the location list remains unresolved, so this step is
not presented as a proven geographic derivation.

Decoding and verification
In CyberChef, applying From Base32 to the corrected sequence yields the flag. Python reproduces the same result:

```
import base64
```

```
encoded = "MNZHKWDJOBUGK4T3KMYFEUSZGRXDA4TUNBJXINDSPU"
padded = encoded + "=" * (-len(encoded) % 8)
decoded = base64.b32decode(padded).decode("utf-8")
print(decoded)
assert base64.b32encode(decoded.encode()).decode().rstrip("=") == encoded
```

Output:

```
cruXipher{S0RRY4n0rthSt4r}
```

Reverse encoding reproduces the corrected sequence exactly, confirming the decoding. It does not prove how the
original addresses yield the intermediate sequence or establish platform acceptance. The final decode is reproducible;
the geographic extraction remains the incomplete part of the solution.

**Source files** santas-nice-list-writeup.md.
