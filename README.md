# Scoreboard Core: In-Order Issue, Out-of-Order Write-Back on SKY130

A small 8-bit processor core built to answer one architecture question on real silicon: how much does out-of-order completion buy over a blocking in-order pipeline, and what does it cost in area?

The design is inspired by the scoreboard in the open-source CVA6 RISC-V core. Instructions issue in program order, functional units with different latencies complete out of order, and a scoreboard commits results back in program order. A mode pin switches the core between a blocking in-order pipeline and the scoreboard pipeline, and on-chip cycle and instruction counters let both modes be measured on the fabricated chip.

Taped out through [Tiny Tapeout](https://tinytapeout.com) on the SkyWater 130nm process (shuttle TTSKY26d).

## Microarchitecture

- 8-bit datapath, 4 architectural registers
- Three functional units with different latencies: 1-cycle ALU, iterative multiplier (~4 cycles), iterative divider (8 cycles)
- 4-entry scoreboard acting as a small reorder buffer for in-order commit
- Rename table tracking the youngest in-flight writer of each register, so WAW hazards do not stall issue
- RAW hazards resolved at issue: operands read from the register file or forwarded from a completed scoreboard entry
- Structural stall when the target functional unit is busy or the scoreboard is full

## Instruction Set

Instructions are 8 bits wide: `[7:6]` opcode, `[5:4]` rd, `[3:2]` rs1, `[1:0]` rs2.

| Opcode | Mnemonic | Operation | Unit |
|---|---|---|---|
| `00` | ADD | rd = rs1 + rs2 | ALU (1 cycle) |
| `01` | MUL | rd = (rs1 × rs2)[7:0] | Multiplier (~4 cycles) |
| `10` | DIV | rd = rs1 / rs2 (divide by zero returns 0xFF) | Divider (8 cycles) |
| `11` | LI | rd = {4'b0, rs1, rs2} (4-bit immediate) | ALU (1 cycle) |

## Operating Modes

| Mode | Behavior |
|---|---|
| 0 | Blocking in-order: an instruction issues only after every older instruction has committed |
| 1 | Scoreboard: independent instructions issue while long-latency operations are still in flight, results complete out of order, commit stays in order |

Running the same instruction stream in both modes and reading the counters gives a direct IPC comparison on silicon.

## Pinout

| Pin | Direction | Function |
|---|---|---|
| `ui_in[7:0]` | in | Instruction |
| `uio[0]` | in | Instruction valid |
| `uio[1]` | in | Mode select (0 = blocking, 1 = scoreboard) |
| `uio[3:2]` | in | Debug select: commit data, cycle counter low, cycle counter high, instructions retired |
| `uio[4]` | out | Ready (low when issue is stalled) |
| `uio[5]` | out | Commit valid |
| `uio[7:6]` | out | Commit destination register |
| `uo_out[7:0]` | out | Commit data, or the counter chosen by debug select |

## Verification

- Python golden reference model of the ISA
- cocotb testbench comparing every commit (order and value) against the model
- Directed tests for RAW stalls, WAW through the rename table, out-of-order completion, full scoreboard, and back-to-back divides
- Constrained-random instruction streams in both modes
- Functional coverage on hazard and completion-order scenarios
- Gate-level simulation on the hardened netlist

## Status

- [x] Specification locked
- [ ] RTL
- [ ] Testbench and reference model
- [ ] Hardening and timing closure
- [ ] Submitted to TTSKY26d (deadline Nov 30, 2026)
- [ ] Silicon bring-up and mode 0 vs mode 1 measurements

## Tools

Verilog, cocotb, Icarus Verilog, Verilator, LibreLane (Yosys + OpenROAD), SkyWater SKY130 PDK.

## Author

Srikanth M G. MS in Computer Engineering, Binghamton University. Digital design and computer architecture.

## About Tiny Tapeout

Tiny Tapeout is an educational project that makes it easier and cheaper than ever to get digital and analog designs manufactured on a real chip. Learn more at [tinytapeout.com](https://tinytapeout.com).
