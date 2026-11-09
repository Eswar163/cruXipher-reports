# JYU



[Original report, page 10](../../cruXipher_CTF_BreakingBad_Report.pdf#page=10)

**Category** Forensics
**Points** 500

The challenge supplied secret.pcap and the hint “a pcap file patented in 1936.” The capture contained approximately
308 loopback packets over 104 seconds, with a single TCP session from 127.0.0.1:56243 to 127.0.0.1:9999.

Following the stream exposed terminal escape sequences, Vim's insert-mode indicator and a new buffer named
flag.txt. The client-to-server one-byte payloads carried keystrokes; terminal query responses had to be excluded.

```
tshark -r secret.pcap -q -z follow,tcp,ascii,0
tshark -r secret.pcap -Y "tcp.srcport==56243 && tcp.len==1" \
  -T fields -e data.data
```

The keys cannot simply be concatenated. The initial i enters insert mode. Later, Escape returns to normal mode, b
moves backward, and cw replaces the final word. Seven backspaces remove part of the replacement, and :wq saves the
buffer. The preserved final text is:

```
jpgQcld.p?k1m{b.y,0pt{j4lygp3+
```

The 1936 clue points to the Dvorak keyboard layout. Mapping each captured Dvorak character to its physical key and
then reading that key as QWERTY produces:

```
cruXipher{v1m_netw0rk_c4ptur3}
```

For example, captured j, p, g, Q map to c, r, u, X; captured ?, {, and + map to {, _, and }. Applying the mapping to the
preserved string confirms the result.

The intermediate buffers contain 21, 18 and 22 characters. The final line has 30 characters, consistent with Vim
reporting 31 bytes written including the newline. The PCAP is not retained, so the packet-level replay cannot be
repeated from this archive. This challenge illustrates why terminal editing state must be reconstructed before applying a
substitution.

Replaying the terminal session

1. Establish the capture format and conversation

```
file secret.pcap
capinfos secret.pcap
tshark -r secret.pcap -q -z io,phs
```

The loopback format and small packet count made a single interactive session plausible. Following TCP stream 0 exposed
terminal screen-clearing sequences and Vim's -- INSERT -- indicator. These identified the application state more
reliably than searching the packet bytes for a flag prefix.

2. Separate terminal replies from user input
The client port, 56243, identifies the direction containing keystrokes. The one-byte filter isolates the typed characters,
but the stream also contains terminal responses such as ESC[2;2R and color-report sequences. Those are replies to
terminal queries, not text intended for flag.txt.

The preserved editing sequence is:

```
i jpgQcld.p?k1m{b.y,prt <ESC>
b cw 0pt{jmfoy.p <BS><BS><BS><BS><BS><BS><BS>
j4lygp3+ <ESC> :wq <CR>
```

3. Track the buffer after each editing operation

```
Operation                  Resulting text                                                Length
```

```
Insert the first string    jpgQcld.p?k1m{b.y,prt                                         21
```

```
Escape, b, cw              jpgQcld.p?k1m{b.y,                                            18
```

```
Insert replacement         jpgQcld.p?k1m{b.y,0pt{jmfoy.p                                 29
```

```
Seven backspaces           jpgQcld.p?k1m{b.y,0pt{                                        22
```

```
Insert final characters    jpgQcld.p?k1m{b.y,0pt{j4lygp3+                                30
```

Here b and cw execute in normal mode. Treating them as literal text would corrupt every subsequent step. The final :wq
is also a command and is excluded from the file content.

4. Convert keyboard positions after reconstructing the text
The mapping must be applied to the saved buffer, after deletions and mode changes. Captured jpgQ becomes cruX; the
shifted keys matter as well, with ? mapping to {, { to _, and + to }. The resulting 30-character flag agrees with the
recorded 31-byte save once the trailing newline is included.

**Source files** JYU-WRITEUP.md.
