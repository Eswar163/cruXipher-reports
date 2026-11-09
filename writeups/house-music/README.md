# House Music



[Original report, page 14](../../cruXipher_CTF_BreakingBad_Report.pdf#page=14)

**Category** Forensics / audio

The input was transmission.wav, recorded as 16-bit stereo PCM at 44100 Hz with 352584 frames. Common noise
obscured the useful signal in either channel alone. Subtracting right from left using signed 32-bit arithmetic cancelled
the shared component without overflowing 16-bit sample values.

A spectrogram of the difference signal revealed 17 Pigpen glyphs. The clue “CRUX first” defines a keyed alphabet by
prepending CRUX and then appending the remaining letters:

```
CRUXABDEFGHIJKLMNOPQSTVWYZ
```

The undotted grid was CRU / XAB / DEF, the dotted grid GHI / JKL / MNO, and the undotted cross top/left/right/bottom
P/Q/S/T. The preserved zero-based indices were:

```
20,18,7,0,21,1,17,18,10,4,20,7,15,4,20,17,16
```

Indexing the keyed alphabet reproduces SPECTROPHASEMASON, which was wrapped as:

```
cruXipher{SPECTROPHASEMASON}
```

The recorded CTFd response for challenge 1 reports status correct. The WAV and plotted glyphs are no longer retained;
the keyed-alphabet conversion remains reproducible. The practical lesson is to compare stereo channels algebraically
before treating noise as irreducible, and to preserve generous plot margins when reading edge glyphs.

Audio recovery steps

1. Confirm that the file contains two usable channels
The original inspection used file, sha256sum and exiftool on transmission.wav. Its 352584 stereo frames at 44100 Hz
represent about eight seconds of audio. The two channels are important: a single-channel conversion would discard the
relationship used to suppress the cover signal.

2. Form the channel difference before plotting
Convert signed 16-bit samples to a wider signed integer type before computing left minus right. If both channels contain
the same noise component N, then (L + N) - (R + N) cancels it. The retained notes identify subtraction, rather than
listening to either channel alone, as the step that exposed the glyphs. They do not retain the exact spectrogram
parameters, so no FFT size or plotting configuration is invented here.

3. Build the keyed Pigpen alphabet
Read the resulting 17 glyphs against the keyed arrangement. “CRUX first” means those four letters occupy the first four
alphabet positions, with duplicates omitted when appending the remaining alphabet. The dotted and undotted shapes
distinguish the two grids; direction distinguishes the cross positions. Using an ordinary A-Z Pigpen layout yields the
wrong substitution.

4. Check the transcription independently of the plot
The preserved indices permit a compact check even though the audio and spectrogram are missing:

```
alphabet = 'CRUXABDEFGHIJKLMNOPQSTVWYZ'
indices = [20,18,7,0,21,1,17,18,10,4,20,7,15,4,20,17,16]
print(''.join(alphabet[i] for i in indices))
```

This prints SPECTROPHASEMASON. It validates the substitution after transcription, while the recorded CTFd acceptance
supports the final wrapped result. The glyph-reading stage still depends on the original spectrogram.

**Source files** combined-writings.md.
