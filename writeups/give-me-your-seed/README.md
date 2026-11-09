# Give me your seed



[Original report, page 25](../../cruXipher_CTF_BreakingBad_Report.pdf#page=25)

**Category** Misc / Minecraft seed search
**Points** 300

The challenge restricted a Minecraft Java Edition 1.21.3 world seed to 100000-150000 and supplied five screenshots
with F3 overlays. Those overlays give coordinates and biomes; visible structures supply additional constraints.

```
Screenshot                 Block coordinates X, Y, Z                                     Biome / clue
```

```
image.png                  -485, 153, -52                                                dark_forest
```

```
image2.png                 2922, 110, -4678                                              dark_forest / mansion
```

```
image3.png                 -1959, 34, 36                                                 desert / portal room
```

```
image4.png                 -1916, 69, 81                                                 desert / pyramid
```

```
image5.png                 -4372, 77, 31                                                 jungle / temple
```

The corrected writeup stresses coordinate signs and flooring: mansion Z -4677.422 maps to block -4678, while portal Z is
positive 36.606. An earlier candidate, 108518, was reported incorrect after coordinate misreadings; it is superseded by
the corrected search.

The C solver uses Cubiomes with MC_1_21_3, iterating all 50001 seeds. For each it checks a biome-valid mansion within
150 blocks of (2922,-4677), a pyramid within 80 of (-1916,81), and a jungle temple within 80 of (-4372,31). It then
prints biomes and stronghold starts for surviving candidates. The tolerances allow for the player's distance from a
generation origin.

The recorded unique survivor is 123321:

```
SEED 123321 mansion 2912,-4624 desert -1920,80 jungle -4368,32
BIOMES 29 29 2 2 21 expected 29 29 2 2 21
stronghold -1964,4
stronghold 964,-1452
stronghold 1076,1588
```

A stronghold start need not coincide with its portal room. Cubiomes checks structure placement and biome eligibility
here, rather than reconstructing every block. The recorded flag is:

```
cruXipher{123321}
```

The source includes a GCC build command linking Cubiomes' noise, biome, layer, generator, finder, utility and quadbase
files. The library is not bundled or pinned, so the search was not rerun in this audit. The five retained screenshots are
reproduced in Appendix B. The key lesson is that coordinate precision and version selection are prerequisites for a
meaningful seed search.

Constraining and testing the seed range

1. Transcribe the F3 overlays precisely
Record the Minecraft version, block position and biome from each screenshot before coding the search. Negative
positions must be floored: -484.956 belongs to block -485. Structure searches use X and Z; the biome checks also retain
the player's Y coordinate. The corrected mansion Z is near -4677, not -467, and the portal room's Z is positive.

2. Match the generation version
The solver defines MC as MC_1_21_3, initializes a generator and applies each candidate seed to the Overworld. This avoids
treating screenshots from one release as constraints for another release's generation rules.

3. Filter using three independent structures
For a target structure, near() obtains its region configuration and searches the neighboring three-by-three region
window. getStructurePos() produces the candidate origin; isViableStructurePos() checks biome eligibility. Squared
distances enforce the mansion, pyramid and temple tolerances without a square-root calculation. A seed is discarded
immediately if any one filter fails.

4. Validate the survivor with unused observations
The structure filters leave seed 123321. The next checks compare all five screenshot biomes and enumerate the first
three stronghold starts. These checks matter because a nearby structure alone could match by chance. The stronghold
start near the portal-room coordinates adds support without requiring the starting staircase and portal room to occupy
the same position.

5. Build and run the retained C solver
The original reproduction instructions use Cubiomes and GCC. From the challenge folder, with the required Cubiomes
source files available:

```
gcc -O3 -fwrapv -Icubiomes solve.c cubiomes/noise.c \
  cubiomes/biomes.c cubiomes/layers.c cubiomes/biomenoise.c \
  cubiomes/generator.c cubiomes/finders.c cubiomes/util.c \
  cubiomes/quadbase.c -lm -o solve.exe
```

The library revision must expose MC_1_21_3; it was not pinned in the source archive. The retained output records one
survivor and matching biome IDs. This procedure verifies placement and biome constraints, not a full visual regeneration
of every screenshot block.

**Source files** Give me your seed/README.md, Give me your seed/solve.c, Give me your seed/images/image.png, Give me your seed/images/image2.png, Give me your seed/images/image3.png, Give me your seed/images/image4.png, Give me your seed/images/image5.png.
