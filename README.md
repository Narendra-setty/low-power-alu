# Low-Power ALU with Clock Gating

A 32-bit Arithmetic Logic Unit (ALU) implemented in Verilog with **integrated clock gating** to minimize dynamic power consumption. Achieves approximately **75% reduction in dynamic power** by selectively disabling clocks to idle functional units using ICG (Integrated Clock Gating) cells.

---

## Overview

Dynamic power in digital circuits is dominated by unnecessary switching activity — even idle logic toggles when a clock is running. This project tackles that at the architectural level by implementing **opcode-based per-unit clock gating**: only the functional unit required for the current operation receives an active clock. All other units are gated off.

This is the same technique used in modern CPUs and SoCs to meet strict power budgets at the hardware level — not just through software power management.

**Key result:** ~75% reduction in dynamic power (P = α·C·V²·f) compared to an ungated baseline ALU.

---

## Architecture

```
         Opcode
            │
            ▼
    ┌───────────────┐
    │  Clock Gate   │  ◀── Master Clock
    │  Controller   │
    └──┬──┬──┬──┬───┘
       │  │  │  │
       ▼  ▼  ▼  ▼
      CK CK CK CK       (gated clocks per unit)
       │  │  │  │
  ┌────┘  │  │  └────┐
  ▼       ▼  ▼       ▼
┌────┐ ┌────┐┌────┐ ┌──────┐
│Arith│ │Logic││Shift│ │Compare│
│Unit│ │Unit││/Rot │ │Unit  │
└────┘ └────┘└────┘ └──────┘
       │  │  │  │
       └──┴──┴──┘
            │
            ▼
         ALU Result
```

### Functional Units

| Unit | Operations | Opcodes |
|---|---|---|
| Arithmetic | ADD, SUB, MUL | `4'b00xx` |
| Logic | AND, OR, XOR, NOT | `4'b01xx` |
| Shift / Rotate | SHL, SHR, ROL, ROR | `4'b10xx` |
| Compare | EQ, NEQ, LT, GT | `4'b11xx` |

### ICG Cell Design

Each unit is gated by a latch-based ICG cell — the industry-standard approach for glitch-free clock gating:

```
Enable ──▶ [Latch] ──▶ [AND] ──▶ Gated Clock
                          ▲
                     Master Clock
```

The latch samples the enable signal on the low phase of the clock, ensuring the gated clock output is glitch-free before the rising edge.

---

## Power Analysis

| Condition | Dynamic Power |
|---|---|
| No clock gating (baseline) | 100% |
| With per-unit clock gating | ~25% |
| **Reduction achieved** | **~75%** |

Power formula: **P = α · C · V² · f**

Where α (activity factor) drops from ~1.0 to ~0.25 when only one of four units is active per cycle.

---

## File Structure

```
low-power-alu/
├── src/
│   ├── alu_top.v           # Top-level ALU module
│   ├── arith_unit.v        # Arithmetic unit
│   ├── logic_unit.v        # Logic unit
│   ├── shift_unit.v        # Shift and rotate unit
│   ├── compare_unit.v      # Compare unit
│   ├── icg_cell.v          # Latch-based ICG cell
│   └── clk_gate_ctrl.v     # Opcode-based clock gate controller
├── tb/
│   └── alu_tb.v            # Self-checking testbench
├── sim/
│   └── waveforms/          # Simulation waveform screenshots
└── docs/
    └── presentation.pptx   # Project presentation
```

---

## Simulation

Simulated and verified in **Vivado**. The self-checking testbench validates all 16 opcodes and verifies that inactive units show zero switching activity when gated.

**To simulate in Vivado:**
1. Create a new project and add all `.v` files from `src/` and `tb/`
2. Set `alu_tb.v` as the top-level simulation source
3. Run Behavioral Simulation
4. Check the Tcl console — all tests print `PASS`

---

## Key Concepts Demonstrated

- **RTL Design** — Multi-module Verilog with clean port interfaces
- **Clock Gating** — Latch-based ICG cell, glitch-free gated clocks
- **Low-Power Design** — Activity factor reduction, dynamic power formula
- **Functional Verification** — Self-checking testbench with full opcode coverage
- **Modular Architecture** — Separate functional units, single top-level integration

---

## Tools

- **Language:** Verilog (RTL)
- **Simulation & Synthesis:** Xilinx Vivado
- **Design Style:** Synchronous, single clock domain with gated clocks per unit

---

## Author

**Narendra Setty** — ECE Student | VLSI & Digital Design | Intern @ IIT Tirupati  
[LinkedIn](https://www.linkedin.com/in/narendrasetty-vlsi) · [GitHub](https://github.com/Narendra-setty)
