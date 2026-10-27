# MIST-ES RISC-V Core: Microchip PolarFire SoC Porting & Architectural Synthesis Analysis

> **Out-of-Context Logic Synthesis & Architectural Mapping** of the MIST-ES RV32IMC core onto the Microchip PolarFire SoC Discovery Kit (MPFS095T).

![Status](https://img.shields.io/badge/Synthesis-PASSED-brightgreen)
![Place & Route](https://img.shields.io/badge/Place%20%26%20Route-Not%20Run-lightgrey)
![Hardware](https://img.shields.io/badge/Board%20Bring--up-Not%20Done-lightgrey)
![ISA](https://img.shields.io/badge/ISA-RV32IMC__Zicsr-blue)

---

## Table of Contents

1. [Scope, Status & Verification Boundary](#1-scope-status--verification-boundary)
2. [Microarchitecture & Configuration](#2-microarchitecture--configuration)
3. [Technology Mapping & Resource Utilization](#3-technology-mapping--resource-utilization)
4. [Static Timing & Critical Path](#4-static-timing--critical-path)
5. [Synthesis Logs, Warnings & Verification](#5-synthesis-logs-warnings--verification)
6. [Reproducibility & Automation](#6-reproducibility--automation)

---

## 1. Scope, Status & Verification Boundary

### Target Summary

| Item | Detail |
|---|---|
| **Target artifact** | `mist_es_core` (standalone core top-level module) |
| **Target silicon** | Microchip PolarFire SoC FPGA (`MPFS095T-FCSG325-1` / `mpfs095tfcvg784-1`) |
| **Board package** | PolarFire SoC Discovery Kit (FCSG325, speed grade -1, core voltage 1.0 V, temp range EXT) |
| **Toolchain** | Microchip Libero SoC v2025.2 / Synopsys Synplify Pro ME (W-2025.03M-SP1-1, Build 183R, Nov 18 2025) |
| **Firmware cross-compiler** | `riscv-none-elf-gcc` (xPack RISC-V GCC 14.2.0), `-march=rv32imc_zicsr -mabi=ilp32 -O2` |
| **Synthesis result** | **PASSED**: Verilog/VHDL Compiler code 0; Premap code 1 (clean); FPGA Mapper code 1 (clean); 0 fatal errors, 93 warnings, 193 notes |

### Verified vs. Unverified

#### ✅ Verified by the synthesis log

- Out-of-context logic synthesis and mapping (`fpga_mapper`) succeeded with **return code 0**: no compile errors, 0 synthesis errors.
- Structural mapping to the PolarFire architecture (MPFS095T logic tiles, arithmetic carry blocks, `MACC_PA` DSP slices, and µSRAM) is complete.
- Utilization counts (**3,865 LUTs, 795 SLEs, 4 MACCs, 0 LSRAMs / 6 µSRAMs**) are exact post-synthesis numbers.
- This is the **Default build**: `PMP_REGIONS = 0` and `U_MODE = 1'b0` are printed in the elaboration log.

#### ❌ NOT yet done

- Place & Route (P&R / layout) has **not** been run in Libero.
- Bitstream generation (`generate_bitstream`) has **not** been run.
- No `.job` / `.stp` bitstream has been downloaded to a physical Discovery board via FlashPro.
- The timing figure (**76.0 MHz**) is a **post-synthesis, pre-layout estimate** (`@N: MT320`: the report is an estimate of place-and-route data; final timing requires the vendor P&R report).

---

## 2. Microarchitecture & Configuration

MIST-ES is an **area-optimized multicycle RISC-V processor** that executes one instruction at a time over a multi-state FSM (no pipeline hazards, no branch forwarding logic). The synthesized instance is the **Baseline Default Configuration**.

### 2.1 Core Subsystems & Parameters

| Subsystem | Details |
|---|---|
| **ISA** | RV32IMC + Zicsr: 32-bit base integer, hardware multiply/divide, 16-bit compressed instructions, CSR access |
| **Privilege** | Machine mode (M-mode) only |
| **`MISA_VAL`** | `32'b0100_0000_0000_0000_0001_0001_0000_0100` (`0x40001104`: RV32, M, C, X) |
| **`U_MODE`** | `1'b0`: user mode deactivated; `mstatus` privilege bits pruned |
| **`PMP_REGIONS`** | `0`: PMP hardware disabled; `pmpaddr_r[0..99]` pruned |
| **Interrupt bits** | `MIE_BIT = 3`, `MPIE_BIT = 7` |
| **Register file** | `mist_es_regfile.sv`: plain 32-entry, dual-read / single-write synchronous array (`WRITE_FIRST = 1'b0`). SECDED ECC variant (`mist_es_regfile_ecc.sv`) **omitted** in this build |
| **Control FSM** | Extracted 8-state primary controller (`state[7:0]`, binary `0000–0111` re-encoded one-hot `00000000 → 10000001`) |
| **Multiply/Divide** | `mist_es_muldiv.sv`: 4-state FSM (`state[3:0]`), 32-bit iterative non-restoring division and hardware DSP multiplication |

---

## 3. Technology Mapping & Resource Utilization

### 3.1 PolarFire SoC Silicon Allocation (`mist_es_core.srr`)

Unlike FPGAs built on 6-input LUTs, PolarFire SoC logic tiles use **4-input LUTs (CFG4)**, dedicated **arithmetic carry cells (ARI1)**, and fine-grained **sequential logic elements (SLE)**.

```text
-------------------------------------------------------------------------------------
Resource Usage Report for mist_es_core (mpfs095tfcvg784-1)
-------------------------------------------------------------------------------------
Total LUTs                   : 3,865
  ├── CFG1 (Inverter/Buffer) : 17 uses
  ├── CFG2 (2-input LUT)     : 318 uses
  ├── CFG3 (3-input LUT)     : 1,045 uses
  └── CFG4 (4-input LUT)     : 1,826 uses
Dedicated Carry Logic        : 659 uses (ARI1: 627 arithmetic, 32 wide-mux)
Sequential Cells (SLE / FF)  : 795 uses
DSP Blocks (MACC_PA)         : 4 of 292 (1%)
  ├── 1 Multiplier (Mult)
  └── 3 Multiplier-Adders (MultAdds cascaded via shift feature)
Block RAM (RAM64x12 Micro-RAM): 6 of 876 (<1%)
Large SRAM (RAM1K20 / LSRAM) : 0 of 308 (0%)
Global Clock Buffers (CLKINT): 1 use
Top-Level I/O Nets           : 359 (Out-of-Context; pad insertion disabled)
-------------------------------------------------------------------------------------
Estimated Extra Logic Required for RAM/MACC Interface during P&R:
  RAM64X12 Interface Logic   : SLEs = 72  ; LUTs = 72
  MACC_PA Interface Logic    : SLEs = 144 ; LUTs = 144
Total Estimated Post-P&R SLEs: 795 + 72 + 144 = 1,011
Total Estimated Post-P&R LUTs: 3,865 + 72 + 144 = 4,081
-------------------------------------------------------------------------------------
```

### 3.2 Architectural Comparison: Artix-7 vs. PolarFire SoC

| Resource | Baseline: Digilent Arty A7 (Vivado 2023.2) | Target: PolarFire Discovery (Synplify Pro ME) | Delta | Justification |
|---|---|---|---|---|
| **Logic tiles** | 2,491 Slice LUTs (LUT6) | 3,865 LUTs (CFG1–4 + ARI1) | **+55.16%** | **LUT-granularity divergence.** AMD 7-series slices have dual-output 6-input LUTs (any 6-input function, or two 5-input functions sharing inputs). PolarFire uses 4-input LUTs (CFG4), so wide muxes (ALU output select, CSR read routing, instruction decode) need two to three cascaded 4-LUTs. |
| **Registers / FFs** | 749 registers | 795 SLEs | **+6.14%** | **State parity with small overhead.** Core state (PC, instruction registers, cycle/instret counters, CSR state) matches closely. The +46 surplus comes from (1) *priority inversion resolution (FX1171)*: PolarFire SLEs prioritize clock enable over reset, so glue logic and register splitting preserve reset-over-enable precedence across CSRs; (2) *timing-driven replication (FX271)*: Synplify cloned 6 registers (incl. `instr[0..3]`, `alu_b[0..4]`) to relieve high-fanout paths. |
| **DSP blocks** | 4 (DSP48E1) | 4 (MACC_PA) | **0.00%** | **Multiplier mapping parity.** The 32×32 multiplier in `mist_es_muldiv.sv` (`prod_full[63:0]`) maps to four 18×18 math slices using hardware cascade shift, matching the DSP48E1 footprint. |
| **Embedded RAM** | 0 (RAMB18/36) | 0 LSRAM (RAM1K20) | **0.00%** | Neither needs large block RAM for the datapath. PolarFire maps the dual-port 32×32 register array (`g_rf.u_rf.regs`) into 6 shallow µSRAM blocks (RAM64x12); Artix-7 uses distributed SliceM LUT-RAM. |

---

## 4. Static Timing & Critical Path

> [!NOTE]
> All figures are **post-synthesis, pre-layout estimates**, not final P&R timing.

| Metric | Value |
|---|---|
| **Clock constraint (inferred)** | 100.0 MHz (T<sub>req</sub> = 10.000 ns) |
| **Worst negative slack (WNS)** | **-3.157 ns** |
| **Estimated F<sub>max</sub>** | **76.00 MHz** (T<sub>est</sub> = 13.157 ns) |
| **Total path delay** | 13.030 ns delay + 0.127 ns setup = 13.157 ns |
| **Logic delay** | 2.401 ns (18.25%) |
| **Interconnect / routing (estimated)** | 10.756 ns (81.75%) |
| **Logic depth** | 16 levels |

### 4.1 Critical Path

The critical vector starts in the **debug controller**, passes through **core decode**, **ALU address calculation**, and **exception dispatch**, and ends at the **Machine Cause Register (`mcause`) enable input**.

**Assessment:** MIST-ES is a multicycle machine with no pipeline stages. Decode, operand fetch, ALU evaluation, and exception checks are evaluated combinationally within specific states. The bottleneck is therefore the chain:

```text
aoff[0] (memory offset calc) → mis (misaligned exception detect) → trap_set → mcause/EN
```

A nominal system clock of **50.0–65.0 MHz** should give positive slack for physical placement and routing without microarchitectural retiming.

---

## 5. Synthesis Logs, Warnings & Verification

Review of the 93 warnings in `mist_es_core.srr` confirms clean synthesis.

| Category | Finding |
|---|---|
| **Unused vector truncation** | `rem_q[32]` in `mist_es_muldiv.sv` (expected overflow guard bit); `instr[19:15]`, `instr[11:7]`, `instr[6:0]` in decoders (standard for formats lacking those fields) |
| **CSR pruning** | `pmpaddr_r[0..99]` pruned to 0 (`PMP_REGIONS = 0`); unimplemented `mstatus` bits (hypervisor / floating-point / user-mode) pruned to 0 |
| **RAM read/write conflict (FX107)** | Flagged on `g_rf.u_rf.regs` and `regs_1`. The multicycle FSM separates read (decode) and writeback (commit) phases, so no concurrent read/write race occurs |
| **I/O over-subscription** | Out-of-context synthesis exposes 359 top-level ports. Pad suppression (`-disable_io_insertion 1`) avoids failing against the Discovery Kit's 80 package pins |

---

## 6. Reproducibility & Automation

The full synthesis flow is automated by a standalone Tcl script, `run_polarfire_disco.tcl`, using paths relative to the repository workspace.

> [!TIP]
> ✅ The script, execution command, and output path below were run and verified in Libero SoC v2025.2 on 2026-10-27.

<details>
<summary><b>Show <code>run_polarfire_disco.tcl</code></b></summary>

```tcl
# ==============================================================================
# Microchip Libero SoC v2025.2 - Standalone Out-of-Context Synthesis Script
# Target: Microchip PolarFire SoC Discovery Kit (MPFS095T-FCSG325-1)
# ==============================================================================

# 1. Project Creation
new_project \
    -location {./mist_pf_proj} \
    -name {mist} \
    -hdl {VERILOG} \
    -family {PolarFireSoC} \
    -die {MPFS095T} \
    -package {FCSG325} \
    -speed {-1} \
    -die_voltage {1.0} \
    -part_range {EXT}

# 2. Source File Import
import_files \
    -convert_EDN_to_HDL 0 \
    -hdl_source {./rtl/common/mist_es_alu.sv} \
    -hdl_source {./rtl/common/mist_es_branch.sv} \
    -hdl_source {./rtl/common/mist_es_csr.sv} \
    -hdl_source {./rtl/common/mist_es_decode.sv} \
    -hdl_source {./rtl/common/mist_es_immgen.sv} \
    -hdl_source {./rtl/common/mist_es_muldiv.sv} \
    -hdl_source {./rtl/common/mist_es_pkg.sv} \
    -hdl_source {./rtl/common/mist_es_pmp.sv} \
    -hdl_source {./rtl/common/mist_es_regfile.sv} \
    -hdl_source {./rtl/common/mist_es_regfile_ecc.sv} \
    -hdl_source {./rtl/common/mist_es_rvc.sv} \
    -hdl_source {./rtl/mist_es_axil.sv} \
    -hdl_source {./rtl/mist_es_core.sv} \
    -hdl_source {./rtl/mist_es_debug.sv} \
    -hdl_source {./rtl/mist_es_soc.sv}

# 3. Hierarchy & Top-Level Module Selection
build_design_hierarchy
set_root -module {mist_es_core::work}

# 4. Out-of-Context Synthesis Execution (Suppress I/O Insertion)
configure_tool -name {SYNTHESIZE} -params {SYNPLIFY_OPTIONS:set_option -disable_io_insertion 1}
run_tool -name {SYNTHESIZE}
```

</details>

### Execution

From PowerShell or Command Prompt at the repository root:

```powershell
& "C:\Microchip\Libero_SoC_2025.2\Libero_SoC\Designer\bin\libero.exe" "SCRIPT:run_polarfire_disco.tcl"
```

### Output

Synthesis log and mapping statistics are generated at:

```text
mist_pf_proj/synthesis/mist_es_core.srr
```
