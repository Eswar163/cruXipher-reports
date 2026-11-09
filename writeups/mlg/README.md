# mlg



[Original report, page 31](../../cruXipher_CTF_BreakingBad_Report.pdf#page=31)

**Category** Pwn
**Points** 400

The target was 134.209.144.148:7834, with a stripped x86-64 PIE executable named chall. The writeup records NX, full
RELRO and no stack canary. It also lists SHSTK/IBT properties; those ELF properties alone do not demonstrate runtime
enforcement, which matters when assessing the documented return-address attack.

Leak and hidden overflow
Menu option 2 leads through the doge dialogue. Answering no to the love question and yes to the kill question prints a
function pointer after AAAAAAAAAAAAA. The archived script subtracts 0x1387 to recover the PIE base. This is a legitimate %p
argument, rather than an uncontrolled format-string vulnerability.

Hidden menu input 420 invokes an 0x80-byte read into a 0x20-byte stack buffer. With no canary, 32 bytes reach saved
RBP and the next eight bytes reach the saved return address.

The normal win function at 0x158c checks a quickscope counter. Entering at 0x15a7 bypasses that gate and starts the
fopen("flag.txt", "r") path. Since this internal entry uses RBP-relative locals without running a prologue, the exploit
supplies a writable fake frame in the mapped data area.

```
base = int(re.search(rb'AAAAAAAAAAAAA(0x[0-9a-f]+)', data).group(1), 16) - DOGE_LEAK
p.send(b'A' * 0x20 + p64(base + SCRATCH) + p64(base + WIN_BODY))
```

The supplied constants are SCRATCH=0x4200 and WIN_BODY=0x15a7. The fake frame leaves room for both the flag buffer at
rbp-0x70 and the file pointer at rbp-8. The source explains stack alignment at the internal entry and notes that the
successful path calls exit, so a complete caller frame is unnecessary.

The recorded remote output contains:

```
cruXipher{mIg_n0_$C0p3_G3t_rek7_m8}
```

The second character of mIg is uppercase I, preserved exactly. The standalone solve.py matches the documented leak
and payload; it was inspected without opening a network connection. A local run also needs flag.txt, which is absent.
The exploit combines a deliberate PIE disclosure, an unbounded read and careful frame placement.

Exploit walkthrough

1. Map the menu and protections

```
file chall
checksec --file=chall
```

PIE requires a runtime code address, NX prevents executing injected stack bytes, and full RELRO rules out the obvious
GOT overwrite. The missing stack canary makes the hidden oversized read the direct route to controlling a return
address.

2. Obtain the runtime code address through option 2
The retained solver waits for otion:, chooses 2, answers no to the doge-love prompt and yes to kil doge?. It extracts the
hexadecimal pointer following AAAAAAAAAAAAA and subtracts DOGE_LEAK = 0x1387. The recorded example gives base
0x55fa356bc000 from leak 0x55fa356bd387.

3. Choose an internal win entry that avoids the guard
The function begins at offset 0x158c, but its counter gate prevents the ordinary route. WIN_BODY = 0x15a7 starts at the
flag-opening path after that check. Entering inside a function requires accounting for the frame setup that was skipped;
otherwise its RBP-relative accesses would use an uncontrolled address.

4. Supply a writable fake frame
The payload uses base + 0x4200 as RBP. The flag buffer is then at base + 0x4190, and the file pointer slot at base +
0x41f8, avoiding overlap. The successful path exits the process after printing, so it does not need a normal caller frame
to return to.

```
Payload field               Length                                                         Purpose
```

```
A padding                   32 bytes                                                       Fill the local buffer
```

```
Packed fake RBP             8 bytes                                                        Provide writable local-variable addresses
```

```
Packed return address       8 bytes                                                        Enter base + 0x15a7
```

5. Trigger option 420 and read the result
The script chooses 420, waits for gamertag:, and sends the 48-byte payload. The vulnerable read permits up to 128 bytes,
so this payload reaches the saved return slot without needing the full limit. On return, the internal win path opens
flag.txt, reads it, prints it and calls exit. The archived remote output is the result evidence; no remote exploit was
rerun while preparing this report.

**Source files** mlg_420blazeit_writeup.md, chall, solve.py.
