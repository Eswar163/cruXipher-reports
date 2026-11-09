# T2F Program



[Original report, page 29](../../cruXipher_CTF_BreakingBad_Report.pdf#page=29)

**Category** Reverse engineering
**Points** 344

t2f-masterclass.zip advertised a single 131072-byte flag.txt, but standard ZIP tools failed to inflate its method-8
stream. The recorded archive size was 121646 bytes, compressed entry size 121532, and CRC32 0xcce1c38d. Zlib
reported invalid literal/lengths set, suggesting a code-tree error before literal decoding.

Parsing the compressed header revealed one dynamic-Huffman DEFLATE block with BFINAL=1, BTYPE=2, HLIT=257,
HDIST=2, and HCLEN=19. Its literal-code Kraft sum was 1.00390625: oversubscribed by 2 * 2**-9, exactly two extra
nine-bit code contributions.

That observation narrows repair to removing two nine-bit symbols. The documented solver tested all C(46,2)=1035
pairs. Setting code lengths for symbols 100 (d) and 113 (q) to zero uniquely yielded the expected 131072 bytes and
stored CRC32. The resulting Kraft sum was 1.0.

```
[+] after repair: Kraft sum = 1.0
[+] inflated 131072 bytes, CRC32 = 0xcce1c38d, match = True
```

The extracted plaintext reportedly contained the flag at byte offset 65525:

```
cruXipher{sighunzips}
```

The recorded response for CTFd challenge 11 reports correct. The search's unique length-and-CRC match is strong
evidence among the tested repairs; CRC32 alone is not a general cryptographic proof of file identity. Since neither
archive nor solver is supplied, those repair results are recorded evidence rather than a new reproduction. This challenge
rewards diagnosing the compressed bitstream instead of repeatedly switching archive extractors.

Repairing the malformed DEFLATE tree

1. Confirm that the failure is inside the compressed stream

```
file t2f-masterclass.zip
zipinfo -v t2f-masterclass.zip
unzip -o t2f-masterclass.zip -d unz
7z x t2f-masterclass.zip
```

The directory metadata identifies one ordinary deflated entry, but both extractors fail with a data error. Python's
zlib.decompress(data, -15) reports invalid literal/lengths set. The negative window parameter selects a raw
DEFLATE stream, so this error points to its Huffman lengths rather than a missing zlib wrapper.

2. Check the code-length budget
For codeword lengths l, a complete binary prefix tree has Kraft sum sum(2**-l) = 1. The decoded literal-length table
gives 1.00390625, an excess of 0.00390625. Since 2 * 2**-9 equals that excess, deleting two nine-bit contributions is a
focused repair hypothesis rather than an arbitrary bit flip.

3. Enumerate the narrow repair space
There are 46 nine-bit candidates and 1035 unordered pairs. For each pair, remove those two symbol lengths, rebuild the
candidate decoding table, and attempt to recover the entry. Reject candidates that fail decoding, have the wrong length,
or disagree with the stored CRC32. The recorded unique survivor removes symbols 100 and 113.

4. Check output before extracting the flag
The accepted repair emits exactly 131072 bytes with CRC32 0xcce1c38d. Only after those checks does a substring search
locate the flag:

```
grep -abo 'cruXipher{[^}]*}' /tmp/opencode_t2f/flag.txt
```

The recorded result begins at offset 65525. No second encoding layer is required; the flag is literal text in the recovered
entry. The repair script and ZIP are absent, so these steps describe the documented search and its checks rather than
supplying a replacement bitstream parser.

**Source files** combined-writings.md.

Pwn
