# PDF Prince



[Original report, page 11](../../cruXipher_CTF_BreakingBad_Report.pdf#page=11)

**Category** Forensics

The artifact was an internal memo whose producer string read SanitizerPro 2.1 - history purged. The recorded
investigation found three %%EOF markers at offsets 7828, 8984 and 9234, plus repeated definitions of objects 1, 3 and 4.
These are consistent with incremental PDF updates: a current cross-reference can supersede an object without
removing its previous bytes.

```
pdfinfo chall.pdf
grep -abo '%%EOF' chall.pdf
pdftoppm -r 150 -png chall.pdf page
```

Two apparent flags were decoys. An uncompressed stream contained flag{nice_grep_but_not_the_flag}. Text
extraction also yielded ctf{copy_paste_is_a_trap}, while rendering showed dashes. The writeup attributes that
disagreement to a misleading ToUnicode map and zero-width spaces.

The latest catalog's optional-content configuration listed object 12, a hidden keyring layer. Its marked-content block
supplied the repeating XOR key L7-b5acb9c6. The first revision of object 4 retained an encrypted hex string. XORing
those bytes with the archived key produced big-endian 16-bit values, beginning 5, 126, 119, 187, 196.

Because the font used /Identity-H and /CIDToGIDMap /Identity, these values identified glyphs. Reading the embedded
GhostMono outlines, rather than trusting the extraction map, gave the original message:

```
cruX{xr3f_r3m3mb3rs_wh4t_y0u_d3l3t3d}
```

The supplied hex, XOR key and glyph table reproduce that string locally; the original PDF and font are unavailable for
independent outline inspection. The source also suggests a cruXipher{...} alternative conditionally. No acceptance
evidence supports that rewrite, so the artifact's short cruX prefix is retained. The lesson is to inspect historical objects,
hidden layers and glyph appearance together when assessing PDF sanitization.

Recovering the deleted content

1. Compare the document structure with its sanitization claim

```
pdfinfo chall.pdf
strings -a chall.pdf | grep -i flag
grep -abo '%%EOF' chall.pdf
grep -abo 'obj' chall.pdf
```

The metadata says history was purged, but three EOF markers and repeated object definitions show that older revisions
still occupy bytes in the file. The first obvious flag string therefore cannot be trusted merely because strings finds it.

2. Inspect both the current page and its text map
Inflate the latest object 4 content stream and compare its text operators with a rendered page. Object 9's ToUnicode
CMap supplies the extracted text, while the actual embedded glyphs determine appearance. This comparison separates
the copy/paste decoy from visible content. ROT13 explains the memo text but does not recover the deleted message.

3. Read the hidden optional-content group
The latest catalog includes /OCGs [12 0 R] and places that group in /OFF. Inspecting object 12 and its marked-content
block reveals KEYRING / layer 7 and the XOR key. Hidden-layer visibility is a display setting; the content remains
available in the file.

4. Decode the first revision of the page stream
The original object 4 contains this hex operand to Tj:

```
4C322D1C351663D939A7361B3753623E61D462A463484C3C2DE0356A63E039FB36473753624861FE6268637D4C352D4335F
C636A39C8363B37B0627761686293633D4C162D69352363E2
```

Convert hex to bytes, XOR with the repeating bytes of L7-b5acb9c6, then group the result into big-endian pairs. The
recovered identifiers are:

```
5,126,119,187,196,87,126,11,183,157,126,11,130,11,130,152,
11,126,125,157,81,75,2,33,157,8,171,119,157,66,11,170,11,33,11,66,128
```

These values are not plaintext character codes. They fall within the embedded font's glyph range and must be
interpreted through its outlines.

5. Read the embedded font rather than its misleading Unicode map
Inflate object 8's FontFile2 stream to recover the TrueType font. With /Identity-H and an identity CID-to-glyph
mapping, each recovered value selects the corresponding outline. The initial IDs 5,126,119,187,196 spell cruX{.
Continuing through the glyph list recovers the full short-prefix flag. This last step is why ordinary PDF text extraction
cannot solve the challenge by itself.

**Source files** PDFprince-writeup.md.
