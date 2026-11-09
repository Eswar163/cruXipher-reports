# Rice Writing



[Original report, page 17](../../cruXipher_CTF_BreakingBad_Report.pdf#page=17)

**Category** Misc / image analysis
**Points** 50 (recorded API value)

The input was a 10000 x 10000 RGB image containing 311 rice grains. The challenge's sorting, occupied-space and
remainder clues pointed to connected-component area rather than visual identification of the grains.

The documented workflow thresholds the grayscale image at 120, measures each foreground component's pixel area,
sorts those areas ascending, and converts area % 256 to a character. Thresholds 100, 120, 140 and 160 reportedly
produced the same 311 components and areas, supporting the segmentation choice.

The first recorded areas were 41026, 41330, 41593, 41825, 42094. Their remainders decode to Bryan. The full
311-character plaintext reportedly included the substring:

```
othercruXipher{riCegOesWiThaNyThinG}personalities
```

The extracted flag was:

```
cruXipher{riCegOesWiThaNyThinG}
```

The notes quote a successful submission to challenge 9 and a follow-up API record with solved_by_me: true, attempts:
1, and value: 50. The original image, areas.json and decode.py are not supplied, so only the first-five-area arithmetic
can be checked locally. The method demonstrates how component measurements can encode text even when the
image looks like an ordinary object arrangement.

Extracting text from component areas

1. Segment the grains at several thresholds
The documented analysis tested grayscale thresholds 100, 120, 140 and 160. All four gave 311 foreground components
with the same areas. That stability matters because an area-encoded message would be damaged by stray pixels,
merged components or a threshold that clips grain boundaries.

2. Measure area rather than position or outline
Connected-component labeling groups touching foreground pixels into individual grains. For each grain, record the
count of occupied pixels. The clue about the space it occupies points to this measurement; the bounding-box area would
include background pixels and is not the quantity used in the solution.

3. Sort before converting measurements to bytes
The source's sorting clue establishes ascending area as the character order. Reading components in scan order would
retain image placement instead and scramble the message. Reduce each sorted area modulo 256 to recover one byte
per grain.

```
Area                      Remainder modulo 256                                         Character
```

```
41026                     66                                                           B
```

```
Area                      Remainder modulo 256                                          Character
```

```
41330                     114                                                           r
```

```
41593                     121                                                           y
```

```
41825                     97                                                            a
```

```
42094                     110                                                           n
```

The first five characters form Bryan, providing an early check that segmentation, ordering and arithmetic agree. The
complete 311-character text contains the flag within a larger sentence, so recovery requires locating the flag substring
rather than wrapping the entire message.

4. Preserve the decoded case
The mixed capitalization in riCegOesWiThaNyThinG comes from the measured bytes. Normalizing its case would change
the answer. The recorded successful submission and solved_by_me response confirm that exact string; the image and full
area list are missing from the present archive.

**Source files** combined-writings.md.
