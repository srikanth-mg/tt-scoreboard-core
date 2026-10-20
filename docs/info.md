## How it works

An 8-bit core with 4 registers that issues instructions in program order, lets
functional units with different latencies complete out of order, and commits
results back in program order through a 4-entry scoreboard.

Functional units: 1-cycle ALU (ADD, LI), iterative multiplier (~4 cycles),
iterative divider (8 cycles). A rename table tracks the youngest in-flight
writer of each register, so WAW hazards do not stall issue. RAW hazards are
resolved at issue by reading the register file or forwarding from a completed
scoreboard entry.

Instruction format on ui_in: [7:6] opcode, [5:4] rd, [3:2] rs1, [1:0] rs2.
Opcodes: 00 ADD, 01 MUL, 10 DIV (divide by zero returns 0xFF), 11 LI
(rd = 4-bit immediate formed from the rs1 and rs2 fields).

The mode pin selects mode 0 (blocking in-order: issue waits until all older
instructions commit) or mode 1 (scoreboard: out-of-order write-back). On-chip
cycle and retired-instruction counters allow the two modes to be compared.

## How to test

1. Reset the design (rst_n low, then high).
2. Set the mode on uio[1].
3. Drive an instruction on ui_in and raise instr_valid (uio[0]) for one clock
   while ready (uio[4]) is high. Hold the instruction while ready is low.
4. Watch commit_valid (uio[5]): when high, uo_out holds the committed value
   and uio[7:6] holds the destination register.
5. Set dbg_sel (uio[3:2]) to read the cycle counter (low/high byte) or the
   retired-instruction counter on uo_out.
6. Run the same program in both modes and compare cycle counts.

## External hardware

None required. Driven by the Tiny Tapeout demo board.
