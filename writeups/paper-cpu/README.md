# Paper CPU



[Original report, page 28](../../cruXipher_CTF_BreakingBad_Report.pdf#page=28)

**Category** Reverse engineering

The challenge asked us to infer an 8-bit CPU from example binaries and input/output pairs, run challenge.bin on input
0x2a, and format RAM[0x40..0x43] as eight lowercase hex digits. The inputs comprised challenge.bin and four example
binaries.

Instructions are documented as opcode/operand byte pairs with a one-byte 0x70 HALT. Low opcode bits select one of
four registers. The recovered groups were:

```
Group                                                                      Recorded semantics
```

```
0x0r                                                                       Load immediate
```

```
0x1r                                                                       Add register selected by operand low bits
```

```
0x2r                                                                       XOR immediate
```

```
0x3r                                                                       Ambiguous register operation
```

```
0x4r                                                                       Conditional branch with signed byte displacement
```

```
0x5r / 0x6r                                                                Store to / load from RAM
```

```
0x70                                                                       Halt
```

Branches use byte displacements relative to the position after the operand. The preserved example checks include
kront: 0x2a -> 0x4a, vexx: 0x01 -> 0xc3, blip: 0x2a -> 0x7e, and snok: 0xff -> 0x55.

Group 0x3r does not occur in the examples. MOV, ADD, OR and XOR candidates reportedly converge on the same target
output for this input. Two decrement-loop blocks bring R0 to zero; the final bytes are 44 56 e6 80. The first byte agrees
with 0x2a ^ 0x6e = 0x44.

```
cruXipher{4456e680}
```

The notes quote a correct response for CTFd challenge 10. They also say SUB/MUL variants were rejected by the
submission oracle, then say only the successful candidate was submitted. Those statements cannot both establish
rejected platform attempts. This report retains the successful response and treats other-op rejection as unresolved
documentation. A correct answer for one input does not prove a unique ISA.

Deriving and checking the instruction set

1. Identify instruction boundaries
The original workflow compared file output and hex dumps for all binaries. challenge.bin has 60 two-byte instructions
followed by a single-byte HALT, giving 121 bytes. The consistent pair structure separates operands from opcodes and
prevents branch targets from being interpreted as instruction numbers.

2. Use the examples as constraints
The low two opcode bits select a register, while the high nibble determines the operation family. Try candidate
semantics against every supplied input/output pair before running the target. Load-immediate, register addition,
immediate XOR, RAM load/store and branching can then be checked together rather than accepted from one
convenient example.

3. Handle signed byte displacements
Branch operands are signed eight-bit offsets relative to the position after the operand. A byte such as 0xea therefore
represents a negative displacement, not a large forward jump. The recorded loop-backs 40 ea and 40 ec return to
instructions 13 and 33. Interpreting these offsets in instruction units instead of byte units would send execution to
different locations.

4. Keep the unobserved opcode family explicit
The examples do not constrain group 0x3r. MOV, ADD, OR and XOR interpretations converge on the same four output
bytes for input 0x2a; that convergence supports the requested answer without uniquely identifying the operation. The
source's contradictory claim about rejected submissions is not needed to establish the accepted output.

5. Read the requested memory window in order
The documented emulator starts with zeroed RAM and R1/R2, runs the two decrement loops, and reads addresses 0x40,
0x41, 0x42 and 0x43. Concatenate 44, 56, e6 and 80 as eight lowercase hex digits. This is a byte sequence, not a
host-endian integer conversion, so reversing the bytes would change the answer.

**Source files** combined-writings.md.
