# Funny Image



[Original report, page 16](../../cruXipher_CTF_BreakingBad_Report.pdf#page=16)

**Category** Misc / image steganography
**Points** 500

The challenge paired a nearly uniform RGBA image with clues about subtraction and reversed coordinates. The image
contained 52 isolated red-channel pixels whose values were rare and differed from their surroundings by six brightness
levels. Each occupied a separate 256 x 256 block, spanning 16 blocks across each of three rows and four blocks in a
fourth row.

The supplied script selects red values occurring fewer than 100 times, finds their coordinates, and sorts points by (y //
256, x // 256). The global coordinates establish reading order; the offsets within each block carry character values.

```
x_offsets = [px % 256 for px, py in points]
y_offsets = [py % 256 for px, py in points]
flag = ''.join(chr(px - py)
               for px, py in zip(x_offsets, reversed(y_offsets)))
```

Reversing the entire y-offset sequence, rather than each row separately, is essential. The first x offset is recorded as 129
and the final y offset as 30: 129 - 30 = 99, or c. The following pairs recover the flag prefix and then the payload:

```
cruXipher{r_rAnDom_Keey_Th4T_i56_vErY_uNCrAcKaBl237}
```

With Pillow and NumPy installed, reproduction uses:

```
python "Funny Image/solve.py" funny-image.png
```

The original PNG is not retained with the solver. A resized preview cannot substitute for it because the decoding
depends on exact pixel coordinates. This is a coordinate encoding, so preserving the original image bytes is necessary for
future verification.

Pixel-decoding steps

1. Use the full-resolution red channel
The original image is 4096 x 4096 RGBA. The solver converts it to a NumPy array and selects channel 0. It does not
average RGB values or inspect transparency. This matters because the anomaly selection is based on exact red-channel
frequencies.

2. Isolate rare values, then preserve their coordinates
The following excerpt is from the retained solver:

```
values, counts = np.unique(pixels, return_counts=True)
y, x = np.where(np.isin(pixels, values[counts < 100]))
points = sorted(zip(x.tolist(), y.tolist()),
                key=lambda p: (p[1] // 256, p[0] // 256))
```

The frequency cutoff removes the large uniform regions and keeps sparse modified pixels. np.where returns row indices
first, so assigning them to y, x avoids transposing the coordinate roles. Sorting by block row and column then recovers
the intended spatial reading order.

3. Separate block placement from character data
Division by 256 locates the block; modulo 256 gives the encoded offset inside it. Only the offsets enter the subtraction.
Using absolute coordinates would mix the layout grid into the character values and introduce multiples of 256.

4. Reverse the y sequence and subtract
The reversal applies to the entire ordered y-offset list. Pair the first x offset with the last y offset, then work inward. The
first pair gives 129 - 30 = 99, followed by the remaining characters. The script asserts the cruXipher{ prefix and closing
brace after decoding. Those format checks are useful sanity checks, but reproducing the result requires the original
image rather than a screenshot of it.

**Source files** Funny Image/writeup.md, Funny Image/solve.py.
