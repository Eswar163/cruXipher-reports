# Wait For It



[Original report, page 13](../../cruXipher_CTF_BreakingBad_Report.pdf#page=13)

**Category** Forensics

The capture contained roughly 8800 packets over 66 seconds, mostly TCP and DNS. Two IRC sessions reached
10.13.37.7:6667, from 192.168.1.23 and 192.168.1.42. Their repeated messages included plausible decoy flags; the
second conversation supplied the useful hint, “the flag is in the timing.”

The investigation focused on server messages in the first conversation. A recorded stream number was 2, but stream
numbers depend on the capture and should be confirmed before reuse.

```
tcp.port == 6667
tcp.stream == 2
```

In Wireshark, “Seconds Since Previous Displayed Packet” made the timing pattern visible. Ordinary intra-burst gaps were
approximately 6-14 ms. Larger pauses clustered around 0.1 s and 0.3 s. Bursts contained about 10-14 or 27-38 messages,
giving a similar short/long ratio.

The recorded interpretation treats a short burst as a Morse dot and a long burst as a dash. Short pauses separate
symbols; long pauses separate letters. Counting the groups yielded:

```
T1M1NG_1S_3V3RYTH1NG
```

The phrase began repeating before the capture ended. Wrapping the recovered payload gives:

```
cruXipher{T1M1NG_1S_3V3RYTH1NG}
```

The capture is missing, so the thresholds, treatment of underscores and full symbol sequence cannot be replayed here.
The technical distinction is that message counts and inter-packet delays carry the payload while application text acts as
cover traffic.

Timing analysis procedure

1. Identify the two IRC sessions
In Wireshark, open Statistics > Conversations > TCP and find the connections to 10.13.37.7:6667. Follow each TCP
stream before assigning it a role. The 192.168.1.23 conversation carries #ctf; the 192.168.1.42 conversation carries
#offtopic. Their visible flags are decoys, but the latter conversation supplies the timing hint.

2. Isolate the relevant rhythm
Return to the #ctf conversation, confirm its stream number, and set the time display to Seconds Since Previous
Displayed Packet. The recorded stream was 2. Examine the server's repeated message groups: the useful signal is the
burst-and-pause pattern, rather than the changing text or total TCP stream size.

3. Classify bursts and pauses separately

```
Observation                Recorded range                                                Interpretation
```

```
Within-burst spacing       6-14 ms                                                       Ordinary packet rhythm
```

```
Short burst                About 10-14 messages                                          Morse dot
```

```
Long burst                 About 27-38 messages                                          Morse dash
```

```
Short pause                About 0.1 s                                                   Continue the same letter
```

```
Long pause                 About 0.3 s                                                   Finish a letter
```

Both burst lengths and pause lengths have approximately a 1:3 ratio. That joint pattern makes Morse a stronger
hypothesis than treating every packet delay as a raw binary bit.

4. Transcribe one complete repetition
Count each burst up to the following large pause, assign a dot or dash, and use the longer pauses to delimit letters. Near
the start, a roughly 35-message burst followed by a long pause gives a single dash, T. The recorded complete payload is
T1M1NG_1S_3V3RYTH1NG; the next repetition begins before the capture ends. The surviving writeup does not retain the full
Morse transcription or underscore handling, so those details cannot be reconstructed further without the PCAP.

**Source files** Waitforit-writeup.md.
