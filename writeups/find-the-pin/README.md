# Find the PIN



[Original report, page 27](../../cruXipher_CTF_BreakingBad_Report.pdf#page=27)

**Category** Reverse engineering
**Points** 348

The challenge supplied pocketflag, a stripped Linux x86-64 ELF accepting a four-character PIN and producing a decoded
output. Mixed Boolean-arithmetic expressions obscure the input constraints, while a SIGTRAP-driven virtual machine
transforms the encrypted data.

The input checks in main at 0x401713 reduce using (a | b) - (a & b) = a ^ b and the corresponding addition identity.
With ASCII input bytes a,b,c,d, the resulting constraints are:

```
a + d = 0x68
b ^ c = 3
a ^ b = 1
c = 0x34
```

Thus c='4', b=0x34 ^ 3='7', a=0x37 ^ 1='6', and d=0x68-0x36='2'. The PIN is 6742; all four equations were checked
locally.

The disassembly identifies a SIGTRAP handler at 0x401176, encrypted bytes at 0x404040, and custom bytecode at
0x401958:

```
a1 17 cc b4 05 cc f0 07
```

A1 applies an input/index-derived XOR, B4 applies subtraction, and F0 advances the output index and loops. Adjusting
the saved instruction pointer lets the handler interpret operands around int3 instructions. The writeup also records
/proc/self/status, breakpoint-byte and timing checks.

The original environment lacked the binary's Nix-store interpreter. The recorded workaround used an installed Linux
dynamic loader:

```
/lib64/ld-linux-x86-64.so.2 /mnt/c/Users/eswar/Downloads/pocketflag 6742
```

Its observed output was cruXipher{0bfusc4t1on_g0D}. The literal flag has 26 characters, consistent with the
documented output length. This path describes the original Linux environment; it is not a native PowerShell command.
The useful reverse-engineering approach is to simplify the constraints before emulating the signal-driven decoder.

Following the input checks and trap handler

1. Separate the PIN constraints from the decoder
The executable's obfuscation makes the arithmetic look more complicated than the checks it enforces. Simplifying the
mixed Boolean-arithmetic expressions first avoids spending time emulating a decoder under a wrong input. All four
variables are ASCII character values, not integers representing the digits themselves.

2. Solve in dependency order
Start with c = 0x34, which fixes the third character as 4. The XOR relation fixes b = 0x37; the second XOR then fixes a =
0x36. Finally use a + d = 0x68 to obtain d = 0x32. Replacing those bytes with decimal digit values would fail the sum and
XOR constraints.

3. Recognize int3 as part of the virtual machine
The SIGTRAP handler reads bytes after trap instructions and changes the saved instruction pointer. This makes the byte
sequence part of an interpreter rather than ordinary straight-line x86 code. Follow the handler's treatment of the A1, B4
and F0 cases: transform the current encrypted byte, update state, and repeat until 26 output bytes have been
processed.

4. Run under the correct loader
The stored interpreter path referenced an unavailable Nix installation. Invoking the binary through
/lib64/ld-linux-x86-64.so.2 bypassed that path in the recorded Linux environment. The verified input is 6742; the
observed 26-character output has the expected event prefix and closing brace. The original binary is not retained, so the
handler analysis and loader invocation are documented from the preserved disassembly notes rather than rerun here.

**Source files** Find the PIN/writeup.md.
