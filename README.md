<h2 align="center"> # Computer System Architecture – CPU Sim Practicals </h2>
<br>
<h3> **Simulating Mano's Basic Computer in CPU Sim 4.0.11** </h3>

| | |
| :--- | :--- |
| Name | Shivam Pal |
| Roll No. | 26570055 |
| Course / Semester | Computer System Architecture (DSC02 / DSC03 / GE2c) |
| College | Ramanujan College, University of Delhi |
| Tool | CPU Sim 4.0.11 (needs Java 8 **with JavaFX**) |

## Contents

| No. | Practical | Program file |
| :--- | :--- | :--- |
| 1 | [Create a machine based on the Basic Computer](#practical-1-create-a-machine-basic-computer-architecture) | `BasicComputer.cpu` |
| 2 | [Create the fetch routine of the instruction cycle](#practical-2-create-the-fetch-routine-of-the-instruction-cycle) | `programs/P03_ADD.a` (test) |
| 3 | [ADD two user-entered numbers](#practical-3-add-operation-on-two-user-entered-numbers) | `programs/P03_ADD.a` |
| 4 | [SUBTRACT two user-entered numbers](#practical-4-subtract-operation-on-two-user-entered-numbers) | `programs/P04_SUBTRACT.a` |
| 5 | [AND, OR, NOT, XOR, NOR, NAND](#practical-5-logical-operations-and-or-not-xor-nor-nand) | `programs/P05_LOGIC.a` |
| 6 | [Memory-reference instructions ADD, LDA, STA, BUN, ISZ](#practical-6-memory-reference-instructions-add-lda-sta-bun-isz) | `programs/P06_MEMORY_REFERENCE.a` |
| 7 | [Register-reference: CLA, CMA, CME, HLT](#practical-7-register-reference-instructions-cla-cma-cme-hlt) | `programs/P07_REGISTER_REF_CLA_CMA_CME_HLT.a` |
| 8 | [Register-reference: INC, SPA, SNA, SZE](#practical-8-register-reference-instructions-inc-spa-sna-sze) | `programs/P08_REGISTER_REF_INC_SPA_SNA_SZE.a` |
| 9 | [Register-reference: CIR, CIL](#practical-9-register-reference-instructions-cir-cil) | `programs/P09_REGISTER_REF_CIR_CIL.a` |
| 10 | [Sum of integers until a negative number](#practical-10-sum-of-integers-until-a-negative-number-is-read) | `programs/P10_SUM_UNTIL_NEGATIVE.a` |
| 11 | [Sum of integers until zero](#practical-11-sum-of-integers-until-zero-is-read) | `programs/P11_SUM_UNTIL_ZERO.a` |


<br>
---

# Practical 1: Create a Machine (Basic Computer Architecture)

**Aim:** To create, in CPU Sim, a machine based on Mano's Basic Computer: its registers, condition bits, memory, microinstructions, instruction fields and machine instructions.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

CPU Sim describes a computer at the register-transfer level. Four kinds of object make up a machine:

| Object | What it is | Where to edit it |
| :--- | :--- | :--- |
| Hardware modules | Registers, condition bits (single bits that can record a carry or halt the machine) and RAM | Modify → Hardware Modules (Ctrl+K) |
| Microinstructions | One elementary register transfer, such as `PC->AR`, `M[AR]->DR`, a test-and-skip or a decode | Modify → Microinstructions (Ctrl+Shift+M) |
| Fetch sequence | Microinstructions that run before every instruction (built in Practical 2) | Modify → Fetch Sequence (Ctrl+Y) |
| Machine instructions | A name, an opcode, a format made of fields, and an *execute sequence* of microinstructions ending in `End` | Modify → Machine Instructions (Ctrl+M) |

Because every instruction is carried out by a stored list of microinstructions, this is a **microprogrammed** control unit.

The Basic Computer is a 16-bit, single-accumulator machine with 4096 words of memory. Its instruction word is laid out as:

```
 15   14 13 12   11 ........................ 0
+---+----------+------------------------------+
| I |  opcode  |           address            |
+---+----------+------------------------------+
Memory-reference : opcode 000-110         (hex 0xxx to 6xxx when I = 0)
Register-reference: opcode 111, I = 0     (hex 7xxx)
Input-output     : opcode 111, I = 1      (hex Fxxx)
```

> **Bit numbering.** CPU Sim counts bits from the **left**: its bit 0 is the most significant bit, which Mano calls bit 15. So Mano's `IR(0-11)` is CPU Sim's `IR` start bit **4**, 12 bits, and Mano's `AC(0)` (the least significant bit) is CPU Sim's `AC` bit **15**. Check *Execute → Options…* that bits are indexed from the left (the default).

How this CPU Sim machine differs from the textbook design: only direct addressing is used (the I bit is always 0), `INP`/`OUT` move a whole signed integer between the console and AC instead of one character through INPR/OUTR, `HLT` *sets* the halt bit S to 1 (CPU Sim stops when a halt bit becomes 1), and interrupt hardware is left out because no practical needs it.

## Procedure

### Step 1 – Start a new machine

*File → New machine* (Ctrl+Shift+N) creates a machine with no hardware.

<img width="1918" height="1138" alt="image" src="https://github.com/user-attachments/assets/014d4f0d-40d5-45d5-8c72-bdfbad10e9e1" />


<br>

### Step 2 – Create the registers

Open *Modify → Hardware Modules* (Ctrl+K), keep *Type of Module* = **Register**, click **New** for each row and type the name and width (initial value 0).

| Register | Width | Role |
| :--- | :--- | :--- |
| AC | 16 | Accumulator |
| DR | 16 | Data register (memory operand) |
| AR | 12 | Address register |
| PC | 12 | Program counter |
| IR | 16 | Instruction register |
| E | 1 | Extended bit: carry out of the adder, rotated by CIR/CIL |
| TMP | 1 | Scratch bit used only inside CIR and CIL |
| S | 1 | Start/stop flip-flop, set by HLT |

<img width="650" height="677" alt="image" src="https://github.com/user-attachments/assets/fd9c45a3-ecaf-4620-8310-f8e67e6d98c7" />
<br>

### Step 3 – Create the condition bits and the RAM

Switch *Type of Module* to **ConditionBit** and add two bits. Tick **halt** only for `halt-S`: when a microinstruction sets that bit the machine stops.

| Condition bit | Register | Bit | Halt |
| :--- | :--- | :--- | :--- |
| carry-E | E | 0 | no |
| halt-S | S | 0 | yes |

<img width="652" height="672" alt="image" src="https://github.com/user-attachments/assets/122664bb-6bc7-424b-a758-adc71f131d7e" />


Switch to **RAM** and add one module: name `M`, length `4096`, cell size `16`. A 16-bit cell makes the memory word-addressed, like the 4096 × 16 memory of the textbook design. Click **OK**.

<img width="652" height="681" alt="image" src="https://github.com/user-attachments/assets/aa5e600c-eadc-4c90-a3af-ed29ae994345" />

<br>

### Step 4 – Create the microinstructions

Open *Modify → Microinstructions* (Ctrl+Shift+M). Pick each type in **Type of Microinstruction**, click **New** per row and fill the columns as below. The names are only labels, but the execute sequences in Step 6 must use the same names.
<br>

**TransferRtoR** – copy bits from one register to another

| name | source | srcStartBit | dest | destStartBit | numBits |
| :--- | :--- | :--- | :--- | :--- | :--- |
| PC->AR | PC | 0 | AR | 0 | 12 |
| IR(0-11)->AR | IR | 4 | AR | 0 | 12 |
| AR->PC | AR | 0 | PC | 0 | 12 |
| DR->AC | DR | 0 | AC | 0 | 16 |
| AC(0)->TMP | AC | 15 | TMP | 0 | 1 |
| AC(15)->TMP | AC | 0 | TMP | 0 | 1 |
| E->AC(15) | E | 0 | AC | 0 | 1 |
| E->AC(0) | E | 0 | AC | 15 | 1 |
| TMP->E | TMP | 0 | E | 0 | 1 |

> `IR(0-11)->AR` must keep **srcStartBit = 4** and **destStartBit = 0**. Starting at bit 0 would copy the opcode bits into AR and every memory-reference instruction would address the wrong word.

<img width="746" height="676" alt="image" src="https://github.com/user-attachments/assets/f164e255-00cb-42f9-b8db-0a8965284135" />

<br>

**MemoryAccess** – read a memory word into a register, or write a register to memory

| name | direction | memory | data | address |
| :--- | :--- | :--- | :--- | :--- |
| M[AR]->IR | read | M | IR | AR |
| M[AR]->DR | read | M | DR | AR |
| AC->M[AR] | write | M | AC | AR |
| DR->M[AR] | write | M | DR | AR |

<img width="748" height="668" alt="image" src="https://github.com/user-attachments/assets/ce5ee0db-62be-4c22-8ef5-7afd8ad32b7c" />

<br>

**Increment** – add a constant to a register

| name | register | delta |
| :--- | :--- | :--- |
| PC+1->PC | PC | 1 |
| DR+1->DR | DR | 1 |
| AC+1->AC | AC | 1 |

<img width="776" height="683" alt="image" src="https://github.com/user-attachments/assets/07eb177d-c19c-4771-b6db-0545baa18fa4" />

<br>

**Arithmetic** – add two registers; the carry goes to E

| name | type | source1 | source2 | destination | carryBit |
| :--- | :--- | :--- | :--- | :--- | :--- |
| AC+DR->AC,E | ADD | AC | DR | AC | carry-E |

<img width="777" height="687" alt="image" src="https://github.com/user-attachments/assets/cb288279-0232-46d7-8506-bc2a8de3545b" />

<br>

**Logical** – bitwise AND and NOT

| name | type | source1 | source2 | destination |
| :--- | :--- | :--- | :--- | :--- |
| AC^DR->AC | AND | AC | DR | AC |
| AC'->AC | NOT | AC | AC | AC |
| E'->E | NOT | E | E | E |

<img width="741" height="667" alt="image" src="https://github.com/user-attachments/assets/da1d111a-fd57-4e8f-a833-13f580ccaa91" />

<br>

**Shift** – move AC by one bit (used inside CIR and CIL)

| name | source | destination | type | direction | distance |
| :--- | :--- | :--- | :--- | :--- | :--- |
| shr AC | AC | AC | logical | right | 1 |
| shl AC | AC | AC | logical | left | 1 |

<img width="772" height="683" alt="image" src="https://github.com/user-attachments/assets/28c1ad24-e36f-4a8a-8a7b-bba82b937dfc" />

<br>

**Set** – load a constant into bits of a register (clears AC or E)

| name | register | start | numBits | value |
| :--- | :--- | :--- | :--- | :--- |
| 0->AC | AC | 0 | 16 | 0 |
| 0->E | E | 0 | 1 | 0 |

<img width="755" height="677" alt="image" src="https://github.com/user-attachments/assets/24d3e76a-b816-4372-ab00-4e5a52e58801" />

<br>

**Test** – compare bits of a register with a value and omit the next *n* microinstructions when the comparison is true. This is how the skip instructions are built.

| name | register | start | numBits | comparison | value | omission |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| if(DR!=0)skip-1 | DR | 0 | 16 | NE | 0 | 1 |
| if(AC(15)!=0)skip-1 | AC | 0 | 1 | NE | 0 | 1 |
| if(AC(15)==0)skip-1 | AC | 0 | 1 | EQ | 0 | 1 |
| if(AC!=0)skip-1 | AC | 0 | 16 | NE | 0 | 1 |
| if(E!=0)skip-1 | E | 0 | 1 | NE | 0 | 1 |

(For the `AC(15)` tests the *start* is 0 because Mano's AC(15), the sign bit, is CPU Sim's bit 0.)

<img width="745" height="670" alt="image" src="https://github.com/user-attachments/assets/824af80e-9f35-4421-be3e-cd37649e1e8b" />

<br>

**Decode** – pick the machine instruction whose opcode matches IR

| name | ir |
| :--- | :--- |
| decode-IR | IR |

<img width="740" height="703" alt="image" src="https://github.com/user-attachments/assets/a92b37bb-882f-48e1-bbcd-6d6514aafc4c" />

<br>

**SetCondBit** – set the halt bit

| name | bit | value |
| :--- | :--- | :--- |
| 1->S(halt) | halt-S | 1 |

<img width="731" height="690" alt="image" src="https://github.com/user-attachments/assets/526df63a-8b63-4845-9db3-f585cc657440" />

<br>

**IO** – read an integer from, or print an integer to, the console

| name | direction | type | buffer | connection |
| :--- | :--- | :--- | :--- | :--- |
| input-int->AC | input | integer | AC | [Console] |
| output-AC->int | output | integer | AC | [Console] |

<img width="736" height="682" alt="image" src="https://github.com/user-attachments/assets/ee360372-e630-4ac5-a9f0-cac4d9e45773" />


That makes **33 microinstructions**. Click **OK**. An `End` microinstruction exists in every machine automatically; it finishes an execute sequence and returns control to the fetch sequence.

<br>

### Step 5 – Create the instruction fields

*Modify → Machine Instructions* (Ctrl+M) → **Edit Fields…** → **New** three times (all *required*, *absolute*, unsigned, default 0):

| Field | Bits | Used for |
| :--- | :--- | :--- |
| op | 4 | opcode digit of memory-reference instructions (0 to 6) |
| addr | 12 | address part of memory-reference instructions |
| opcode | 16 | the complete 16-bit code of register-reference and I/O instructions |

<img width="522" height="607" alt="image" src="https://github.com/user-attachments/assets/444964ec-8041-477f-8641-523f9f8ddec7" />


> **Why two opcode lengths work.** The decoder compares the leftmost 1 bit of IR, then 2 bits, then 3 and so on, and picks the first instruction whose opcode matches. No instruction has the 4-bit opcode 7 or F, so an IR such as `7800` keeps being examined until all 16 bits match `CLA`.

<br>

### Step 6 – Create the 20 machine instructions

For each instruction click **new**, type the name, then on the **Format** tab enter the opcode and drag the fields into the format bar (`op` then `addr`, or just `opcode`). On the **Implementation** tab drag the microinstructions into *Execute sequence* in the order shown, ending with `End`.

| Name | Opcode | Format | Execute sequence |
| :--- | :--- | :--- | :--- |
| AND | 0x0 | op addr | M[AR]->DR, AC^DR->AC, End |
| ADD | 0x1 | op addr | M[AR]->DR, AC+DR->AC,E, End |
| LDA | 0x2 | op addr | M[AR]->DR, DR->AC, End |
| STA | 0x3 | op addr | AC->M[AR], End |
| BUN | 0x4 | op addr | AR->PC, End |
| ISZ | 0x6 | op addr | M[AR]->DR, DR+1->DR, DR->M[AR], if(DR!=0)skip-1, PC+1->PC, End |
| CLA | 0x7800 | opcode | 0->AC, End |
| CLE | 0x7400 | opcode | 0->E, End |
| CMA | 0x7200 | opcode | AC'->AC, End |
| CME | 0x7100 | opcode | E'->E, End |
| CIR | 0x7080 | opcode | AC(0)->TMP, shr AC, E->AC(15), TMP->E, End |
| CIL | 0x7040 | opcode | AC(15)->TMP, shl AC, E->AC(0), TMP->E, End |
| INC | 0x7020 | opcode | AC+1->AC, End |
| SPA | 0x7010 | opcode | if(AC(15)!=0)skip-1, PC+1->PC, End |
| SNA | 0x7008 | opcode | if(AC(15)==0)skip-1, PC+1->PC, End |
| SZA | 0x7004 | opcode | if(AC!=0)skip-1, PC+1->PC, End |
| SZE | 0x7002 | opcode | if(E!=0)skip-1, PC+1->PC, End |
| HLT | 0x7001 | opcode | 1->S(halt), End |
| INP | 0xF800 | opcode | input-int->AC, End |
| OUT | 0xF400 | opcode | output-AC->int, End |

Reading the skip instructions: the test microinstruction omits `PC+1->PC` when the *opposite* condition holds, so PC is incremented (the next instruction is skipped) only when the instruction's own condition is true.

<img width="882" height="675" alt="image" src="https://github.com/user-attachments/assets/43d01af0-4c58-4470-a644-8b6e35d96131" />

<br>

<img width="862" height="592" alt="image" src="https://github.com/user-attachments/assets/51cc55bd-9db6-4e29-b399-518acf218854" />

<br>

<img width="863" height="608" alt="image" src="https://github.com/user-attachments/assets/61ec07c8-3a4d-4358-a941-86e17b442239" />


<img width="857" height="625" alt="image" src="https://github.com/user-attachments/assets/7c1eb682-fe2b-480c-ab11-acde9eeb2a94" />


### Step 7 – Fetch sequence, program counter and saving

Build the fetch sequence as described in Practical 2. Then in *Execute → Options…* choose **PC** as the program counter (debug mode needs it). Finally *File → Save machine as…* → `BasicComputer.cpu`.

## Observations

The Hardware Modules dialog shows 8 registers, 2 condition bits and the 4096-word RAM `M`. The machine has 33 microinstructions and 20 machine instructions. Loading `P03_ADD.a` (Practical 3) gives the machine code `F800 3007 F800 1007 …` in RAM, which confirms the instruction formats.

## Result

A machine based on the Basic Computer architecture was created in CPU Sim and saved as `BasicComputer.cpu`.

## Viva questions

**Q1. Why are PC and AR 12 bits wide while AC is 16 bits?**
Memory holds 4096 = 2¹² words, so an address needs 12 bits; each data word is 16 bits.

**Q2. Machine instruction versus microinstruction?**
A machine instruction (e.g. `ADD`) is what the programmer writes; it is carried out by a list of microinstructions, each a single register transfer such as `DR ← M[AR]`.

**Q3. What does the *halt* option of a condition bit do?**
When a microinstruction sets that bit to 1, CPU Sim stops the program. `HLT` uses it through `halt-S`.

**Q4. Why is the RAM cell size 16?**
Each instruction or data value is one 16-bit word, so one address holds one word and PC can simply advance by 1.

**Q5. Is this control unit hardwired or microprogrammed?**
Microprogrammed: every instruction is a stored sequence of microinstructions.

---

<br>

# Practical 2: Create the Fetch Routine of the Instruction Cycle

**Aim:** To create the fetch (and decode) routine of the instruction cycle and watch it run one microinstruction at a time.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

Every instruction cycle starts with the same fetch and decode phase. In Mano's design it takes three clock pulses, driven by the sequence counter outputs T0, T1 and T2:

```
T0 : AR <- PC
T1 : IR <- M[AR],  PC <- PC + 1
T2 : D0..D7 <- decode IR(12-14),  AR <- IR(0-11),  I <- IR(15)
```

After T2 the control unit knows which instruction is in IR and AR already holds its address field, so execution starts at T3. In CPU Sim the fetch routine is a list of microinstructions that runs before every execute sequence. Real hardware does `IR ← M[AR]` and `PC ← PC + 1` on the same clock edge; CPU Sim runs microinstructions one after another, so they appear as separate steps.

| Order | CPU Sim microinstruction | Register transfer | Time step |
| :--- | :--- | :--- | :--- |
| 1 | `PC->AR` | AR ← PC | T0 |
| 2 | `M[AR]->IR` | IR ← M[AR] | T1 |
| 3 | `PC+1->PC` | PC ← PC + 1 | T1 |
| 4 | `IR(0-11)->AR` | AR ← IR(0–11) | T2 |
| 5 | `decode-IR` | D0…D7 ← decode IR(12–14) | T2 |

## Procedure

1. Open the machine from Practical 1 and choose *Modify → Fetch Sequence* (Ctrl+Y).
2. In the *MicroInstructions* tree, drag `PC->AR` (transferRtoR) into *Fetch Sequence Implementation*.
3. Drag `M[AR]->IR` (memoryAccess) below it.
4. Drag `PC+1->PC` (increment).
5. Drag `IR(0-11)->AR` (transferRtoR).
6. Drag `decode-IR` (decode) to the end, click **OK**, and save the machine (Ctrl+B).

<img width="630" height="593" alt="image" src="https://github.com/user-attachments/assets/f951df6c-98a7-4d81-90a8-338ffea88da6" />


## Testing the routine

Open `programs/P03_ADD.a`, press **Ctrl+2** to assemble and load, then **Ctrl+D** for debug mode. Set the Registers *Data* box to **Unsigned Dec**. Click **Step by Micro** five times and watch which register each microinstruction changes (changed registers are outlined in green).

<img width="1918" height="1132" alt="image" src="https://github.com/user-attachments/assets/3f5e8802-82ce-494f-bee4-68bb175ecbc3" />

<br>

<img width="1918" height="1133" alt="image" src="https://github.com/user-attachments/assets/b9654842-9a72-4215-bd4e-c90b7da124a6" />

<img width="1917" height="1132" alt="image" src="https://github.com/user-attachments/assets/e2802997-6f4d-4d1c-83b3-d2c249c70d52" />



## Observations

| Micro-step | Microinstruction | AR | PC | IR |
| :--- | :--- | :--- | :--- | :--- |
| **start** | -- | 0 | 0 | 0 |
| **1** | `PC->AR` | 0 | 0 | 0 |
| **2** | `M[AR]->IR` | 0 | 0 | 63488 (F800) |
| **3** | `PC+1->PC` | 0 | 1 | 63488 (F800) |
| **4** | `IR(0-11)->AR` | 2048 (800) | 1 | 63488 (F800) |
| **5** | `decode-IR` | 2048 (800) | 1 | 63488 (F800) → INP |

The first instruction in memory is `F800` (`INP`). `IR(0-11)->AR` copies its low 12 bits (`800` hex = 2048) into AR, even though INP does not use AR as an address; the fetch routine is shared by all instructions.

## Result

The fetch routine `PC->AR`, `M[AR]->IR`, `PC+1->PC`, `IR(0-11)->AR`, `decode-IR` was created and verified by single-stepping the first instruction of a program.

## Viva questions

**Q1. Why is PC incremented during fetch?**
So it already points to the next instruction; a branch can simply overwrite it and a skip can add one more.

**Q2. Why does AR receive IR(0–11) even for register-reference instructions?**
The fetch routine is common to every instruction. For memory-reference instructions that field is the operand address; for the others it is simply unused.

**Q3. What does the decode microinstruction do?**
It examines IR and transfers control to the execute sequence of the matching machine instruction.

**Q4. How many clock pulses does fetch-and-decode take in the Basic Computer?**
Three: T0, T1 and T2.

---

# Practical 3: ADD Operation on Two User-entered Numbers

**Aim:** To write an assembly program that reads two numbers entered by the user, adds them and displays the sum.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

`INP` reads an integer into AC. Because the second `INP` overwrites AC, the first number is saved in memory with `STA A`. `ADD A` is a memory-reference instruction: `DR ← M[A]`, then `AC ← AC + DR`, and the carry out of the top bit goes to E. `OUT` prints AC and `HLT` stops the machine.

Numbers are 16-bit two's complement, so the range is −32768 to +32767. A sum outside that range wraps around (see the last sample run below).

## Program – `programs/P03_ADD.a`

```
; ==============================================================
; Practical 3 : ADD two numbers typed by the user
; Machine     : BasicComputer.cpu (Mano's Basic Computer)
; Idea        : SUM = A + B
; ==============================================================
        INP             ; AC <- first number from the console
        STA A           ; keep it in memory, the next INP overwrites AC
        INP             ; AC <- second number
        ADD A           ; AC <- AC + M[A]   (E <- carry out of bit 15)
        STA SUM         ; store the result
        OUT             ; print AC
        HLT             ; stop the machine
A:      .data 1 0       ; first number
SUM:    .data 1 0       ; result
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | F800 |  | INP |
| 1 | 001 | 3007 |  | STA A |
| 2 | 002 | F800 |  | INP |
| 3 | 003 | 1007 |  | ADD A |
| 4 | 004 | 3008 |  | STA SUM |
| 5 | 005 | F400 |  | OUT |
| 6 | 006 | 7001 |  | HLT |
| 7 | 007 | 0000 | A | .data 1 0 |
| 8 | 008 | 0000 | SUM | .data 1 0 |

Memory-reference instructions are the opcode digit followed by the 12-bit address (`ADD A` with A at address 7 is `1007`); `INP`, `OUT` and `HLT` are fixed 16-bit codes (`F800`, `F400`, `7001`).

## Procedure

1. *File → Open machine…* → `BasicComputer.cpu`.
2. *File → Open text…* → `P03_ADD.a`. Set the RAM pane's *Data* box to **Hex**.
3. Press **Ctrl+2** (assemble and load) and compare the RAM pane with the memory map above.
4. Press **Ctrl+R**. When the console turns yellow type `25` and press Enter, then type `17` and press Enter.
5. The console prints `Output: 42` and the halt message.

<img width="1918" height="1133" alt="image" src="https://github.com/user-attachments/assets/eb49cab6-f2ab-41c1-9e6b-ed9ab3574467" />

<img width="1918" height="1127" alt="image" src="https://github.com/user-attachments/assets/8e9519d7-b5b1-491a-a913-35fde8a1d255" />

<img width="1918" height="1133" alt="image" src="https://github.com/user-attachments/assets/0114e8b8-7460-44c5-9005-c4f8d0d54a0f" />


## Observations

| Input(s) typed | Output(s) displayed |
| :--- | :--- |
| 25, 17 | 42 |
| -40, 15 | -25 |
| -1, 1 | 0 |
| 30000, 10000 | -25536 |

With inputs 25 and 17 the final registers are AC = 42, DR = 25 (the operand read by `ADD`), PC = 7, IR = 28673 (`7001`, the HLT) and E = 0. Memory holds A = `0019` and SUM = `002A` (hex).

For −1 + 1 the result is 0 and **E = 1**: the 16-bit addition `FFFF + 0001` carries out of bit 15. For 30000 + 10000 the true sum 40000 does not fit in 16-bit two's complement, so the output wraps to 40000 − 65536 = −25536.

## Result

The program correctly adds two user-entered numbers: 25 + 17 = **42**.

## Viva questions

**Q1. Why is the first number stored before the second `INP`?**
`INP` overwrites AC, so the first number would be lost.

**Q2. What is the machine code of `ADD A` if A is at address 7?**
Opcode 1 and address 007: `1007` hex.

**Q3. What does E hold after `ADD`?**
The carry out of the most significant bit of the addition.

**Q4. Why does 30000 + 10000 print −25536?**
The sum exceeds +32767; the 16-bit result `9C40` is read as a negative two's complement number (overflow).

---

# Practical 4: SUBTRACT Operation on Two User-entered Numbers

**Aim:** To write an assembly program that reads two numbers A and B and displays A − B.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

The Basic Computer has no subtract instruction, so subtraction is done with the two's complement: **A − B = A + (B′ + 1)**. `CMA` forms the one's complement B′, `INC` adds 1 to give −B, and `ADD` adds A.

```
A       = 18 = 0000 0000 0001 0010
B       = 50 = 0000 0000 0011 0010
B'      =      1111 1111 1100 1101     (CMA)
B' + 1  =      1111 1111 1100 1110     (INC)  = -50
A + (-B)=      1111 1111 1110 0000     = -32
```

## Program – `programs/P04_SUBTRACT.a`

```
; ==============================================================
; Practical 4 : SUBTRACT two numbers typed by the user (A - B)
; Machine     : BasicComputer.cpu (Mano's Basic Computer)
; Idea        : there is no SUB instruction, so
;               A - B = A + (B' + 1)   (two's complement of B)
; ==============================================================
        INP             ; AC <- A (minuend)
        STA A           ; save A
        INP             ; AC <- B (subtrahend)
        CMA             ; AC <- B'      (one's complement)
        INC             ; AC <- B' + 1  (= -B)
        ADD A           ; AC <- -B + A
        STA DIFF        ; store the difference
        OUT             ; print AC
        HLT
A:      .data 1 0       ; minuend
DIFF:   .data 1 0       ; result
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | F800 |  | INP |
| 1 | 001 | 3009 |  | STA A |
| 2 | 002 | F800 |  | INP |
| 3 | 003 | 7200 |  | CMA |
| 4 | 004 | 7020 |  | INC |
| 5 | 005 | 1009 |  | ADD A |
| 6 | 006 | 300A |  | STA DIFF |
| 7 | 007 | F400 |  | OUT |
| 8 | 008 | 7001 |  | HLT |
| 9 | 009 | 0000 | A | .data 1 0 |
| 10 | 00A | 0000 | DIFF | .data 1 0 |

## Procedure

1. Open `BasicComputer.cpu` and `P04_SUBTRACT.a`; set the RAM *Data* box to **Hex**.
2. Press **Ctrl+2**, then **Ctrl+R**.
3. Enter the minuend `18` (Enter), then the subtrahend `50` (Enter).
4. The console prints `Output: -32`.

<img width="1918" height="1042" alt="image" src="https://github.com/user-attachments/assets/10236904-8a02-426d-a78a-4c5c99459dd4" />


<img width="1918" height="1122" alt="image" src="https://github.com/user-attachments/assets/14c54066-c5e0-4ca1-8bbe-d61d2549af20" />


<img width="1918" height="1132" alt="image" src="https://github.com/user-attachments/assets/161f97b3-6a5f-4c4f-9aa5-33ff05294c9b" />


## Observations

| Input(s) typed | Output(s) displayed |
| :--- | :--- |
| 50, 18 | 32 |
| 18, 50 | -32 |
| -7, -7 | 0 |
| 0, 1 | -1 |

For 18 − 50 the final AC is −32 (`FFE0` hex), DR = 18 and PC = 9.

## Result

The program subtracts two user-entered numbers using the two's complement: 18 − 50 = **−32** and 50 − 18 = **32**.

## Viva questions

**Q1. How do you subtract without a SUB instruction?**
Add the two's complement of the subtrahend: complement it (`CMA`), add 1 (`INC`), then `ADD`.

**Q2. Why is `INC` needed after `CMA`?**
`CMA` gives −B − 1; adding 1 gives −B.

**Q3. What is the two's complement of `0000 0000 0011 0010`?**
`1111 1111 1100 1110` (`FFCE` hex) = −50.

**Q4. Which register-reference instructions does the program use?**
`CMA` (7200), `INC` (7020) and `HLT` (7001).

---

# Practical 5: Logical Operations: AND, OR, NOT, XOR, NOR, NAND

**Aim:** To write an assembly program that performs AND, OR, NOT, XOR, NOR and NAND on two user-entered numbers.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

The machine has only two logic instructions: `AND` (memory-reference, AC ← AC ∧ M[x]) and `CMA` (AC ← AC′). Since {AND, NOT} is functionally complete, everything else can be built from them with Boolean algebra, applied to all 16 bits at once:

| Operation | Identity used | Instruction sequence |
| :--- | :--- | :--- |
| A AND B | A·B | `LDA A`, `AND B` |
| A OR B | (A′·B′)′ (De Morgan) | `LDA B`, `CMA`, `STA NB`, `LDA A`, `CMA`, `AND NB`, `CMA` |
| NOT A | A′ | `LDA A`, `CMA` |
| A XOR B | (A + B)·(A·B)′ | `LDA RAND`, `CMA`, `AND ROR` |
| A NOR B | (A + B)′ | `LDA ROR`, `CMA` |
| A NAND B | (A·B)′ | `LDA RAND`, `CMA` |

Results are shown as signed decimal numbers. Complementing a small positive number sets the sign bit, so NOT 12 = −13 (in two's complement, x′ = −x − 1).

## Program – `programs/P05_LOGIC.a`

```
; ==============================================================
; Practical 5 : AND, OR, NOT, XOR, NOR, NAND on two numbers A and B
; Machine     : BasicComputer.cpu (Mano's Basic Computer)
;
; Only two logic instructions exist: AND (AC <- AC . M[x]) and CMA
; (AC <- AC'). {AND, NOT} is functionally complete, so:
;     OR   = (A' . B')'          (De Morgan)
;     NAND = (A . B)'
;     NOR  = (A + B)'
;     XOR  = (A + B) . (A . B)'
; Output order: AND, OR, NOT A, NOT B, XOR, NOR, NAND
; ==============================================================
        INP             ; AC <- A
        STA A
        INP             ; AC <- B
        STA B
; ----- AND = A . B ------------------------------------------
        LDA A
        AND B           ; AC <- A . B
        STA RAND
        OUT             ; output 1
; ----- OR = (A' . B')' --------------------------------------
        LDA B
        CMA             ; AC <- B'
        STA NB
        LDA A
        CMA             ; AC <- A'
        STA NA
        AND NB          ; AC <- A' . B'
        CMA             ; AC <- A + B
        STA ROR
        OUT             ; output 2
; ----- NOT A and NOT B --------------------------------------
        LDA NA
        OUT             ; output 3
        LDA NB
        OUT             ; output 4
; ----- XOR = (A + B) . (A . B)' -----------------------------
        LDA RAND
        CMA             ; AC <- (A . B)'
        STA RNAND
        AND ROR         ; AC <- (A + B) . (A . B)'
        STA RXOR
        OUT             ; output 5
; ----- NOR = (A + B)' ---------------------------------------
        LDA ROR
        CMA
        STA RNOR
        OUT             ; output 6
; ----- NAND = (A . B)' --------------------------------------
        LDA RNAND
        OUT             ; output 7
        HLT
A:      .data 1 0
B:      .data 1 0
NA:     .data 1 0       ; A'
NB:     .data 1 0       ; B'
RAND:   .data 1 0       ; A AND B
ROR:    .data 1 0       ; A OR B
RXOR:   .data 1 0       ; A XOR B
RNOR:   .data 1 0       ; A NOR B
RNAND:  .data 1 0       ; A NAND B
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | F800 |  | INP |
| 1 | 001 | 3023 |  | STA A |
| 2 | 002 | F800 |  | INP |
| 3 | 003 | 3024 |  | STA B |
| 4 | 004 | 2023 |  | LDA A |
| 5 | 005 | 0024 |  | AND B |
| 6 | 006 | 3027 |  | STA RAND |
| 7 | 007 | F400 |  | OUT |
| 8 | 008 | 2024 |  | LDA B |
| 9 | 009 | 7200 |  | CMA |
| 10 | 00A | 3026 |  | STA NB |
| 11 | 00B | 2023 |  | LDA A |
| 12 | 00C | 7200 |  | CMA |
| 13 | 00D | 3025 |  | STA NA |
| 14 | 00E | 0026 |  | AND NB |
| 15 | 00F | 7200 |  | CMA |
| 16 | 010 | 3028 |  | STA ROR |
| 17 | 011 | F400 |  | OUT |
| 18 | 012 | 2025 |  | LDA NA |
| 19 | 013 | F400 |  | OUT |
| 20 | 014 | 2026 |  | LDA NB |
| 21 | 015 | F400 |  | OUT |
| 22 | 016 | 2027 |  | LDA RAND |
| 23 | 017 | 7200 |  | CMA |
| 24 | 018 | 302B |  | STA RNAND |
| 25 | 019 | 0028 |  | AND ROR |
| 26 | 01A | 3029 |  | STA RXOR |
| 27 | 01B | F400 |  | OUT |
| 28 | 01C | 2028 |  | LDA ROR |
| 29 | 01D | 7200 |  | CMA |
| 30 | 01E | 302A |  | STA RNOR |
| 31 | 01F | F400 |  | OUT |
| 32 | 020 | 202B |  | LDA RNAND |
| 33 | 021 | F400 |  | OUT |
| 34 | 022 | 7001 |  | HLT |
| 35 | 023 | 0000 | A | .data 1 0 |
| 36 | 024 | 0000 | B | .data 1 0 |
| 37 | 025 | 0000 | NA | .data 1 0 |
| 38 | 026 | 0000 | NB | .data 1 0 |
| 39 | 027 | 0000 | RAND | .data 1 0 |
| 40 | 028 | 0000 | ROR | .data 1 0 |
| 41 | 029 | 0000 | RXOR | .data 1 0 |
| 42 | 02A | 0000 | RNOR | .data 1 0 |
| 43 | 02B | 0000 | RNAND | .data 1 0 |

## Procedure

1. Open `BasicComputer.cpu` and `P05_LOGIC.a`; set the RAM *Data* box to **Hex**.
2. Press **Ctrl+2**, then **Ctrl+R**; enter `12` (Enter) and `10` (Enter).
3. Seven outputs are printed in the order AND, OR, NOT A, NOT B, XOR, NOR, NAND. Scroll the console up to see them all.

<img width="1918" height="1126" alt="image" src="https://github.com/user-attachments/assets/d2844e24-3727-4e1b-838d-f7674eefea04" />


<img width="1918" height="1135" alt="image" src="https://github.com/user-attachments/assets/f4a3011c-23fe-4e8f-9a81-18d17c3bf74e" />


<img width="1918" height="1127" alt="image" src="https://github.com/user-attachments/assets/0e9913a2-5218-4bee-8bbf-da5f44e294d0" />


## Observations

For A = 12 and B = 10:

| Operation | 16-bit result (binary) | Hex | Decimal output |
| :--- | :--- | :--- | :--- |
| A = 12 | 0000 0000 0000 1100 | 000C | 12 |
| B = 10 | 0000 0000 0000 1010 | 000A | 10 |
| A AND B | 0000 0000 0000 1000 | 0008 | 8 |
| A OR B | 0000 0000 0000 1110 | 000E | 14 |
| NOT A | 1111 1111 1111 0011 | FFF3 | -13 |
| NOT B | 1111 1111 1111 0101 | FFF5 | -11 |
| A XOR B | 0000 0000 0000 0110 | 0006 | 6 |
| A NOR B | 1111 1111 1111 0001 | FFF1 | -15 |
| A NAND B | 1111 1111 1111 0111 | FFF7 | -9 |

Further runs:

| Input(s) typed | Output(s) displayed (AND, OR, NOT A, NOT B, XOR, NOR, NAND) |
| :--- | :--- |
| 12, 10 | 8, 14, -13, -11, 6, -15, -9 |
| 5, 3 | 1, 7, -6, -4, 6, -8, -2 |
| 255, 15 | 15, 255, -256, -16, 240, -256, -16 |

## Result

All six logical operations were produced using only `AND` and `CMA`. For A = 12 and B = 10 the outputs are AND = 8, OR = 14, NOT A = −13, NOT B = −11, XOR = 6, NOR = −15, NAND = −9.

## Viva questions

**Q1. Which logic instructions does the Basic Computer have?**
`AND` (memory-reference) and `CMA` (complement AC). OR and XOR have to be built from them.

**Q2. State De Morgan's law as used for OR.**
A + B = (A′·B′)′.

**Q3. Why is NOT 12 displayed as −13?**
`CMA` flips all 16 bits: `000C` becomes `FFF3`, which is −13 in two's complement.

**Q4. What does "functionally complete" mean?**
Every Boolean function can be written using only that set of operations, e.g. {AND, NOT} or {NAND}.

**Q5. How would you clear the upper 8 bits of AC?**
`AND` with a memory word containing `00FF` (a mask).

---

# Practical 6: Memory-reference Instructions: ADD, LDA, STA, BUN, ISZ

**Aim:** To write an assembly program that exercises the memory-reference instructions `ADD`, `LDA`, `STA`, `BUN` and `ISZ`, and record the registers after every instruction.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

A memory-reference instruction has an opcode 0–6 and a 12-bit address. During fetch `AR ← IR(0–11)`, so when execution starts AR already holds the operand address (direct addressing, I = 0).

| Symbol | Code | Execute micro-operations |
| :--- | :--- | :--- |
| ADD | 1xxx | DR ← M[AR]; AC ← AC + DR, E ← Cout |
| LDA | 2xxx | DR ← M[AR]; AC ← DR |
| STA | 3xxx | M[AR] ← AC |
| BUN | 4xxx | PC ← AR |
| ISZ | 6xxx | DR ← M[AR]; DR ← DR + 1; M[AR] ← DR; if DR = 0 then PC ← PC + 1 |

The program multiplies X = 5 by N = 3 using repeated addition. `CTR` starts at −3. Each pass adds X to `PROD` and `ISZ CTR` increments CTR; while CTR is not zero the following `BUN LOOP` repeats the loop. When CTR reaches 0 the `ISZ` skips the `BUN` and the program ends with PROD = 15.

## Program – `programs/P06_MEMORY_REFERENCE.a`

```
; ==============================================================
; Practical 6 : memory-reference instructions ADD, LDA, STA, BUN, ISZ
; Machine     : BasicComputer.cpu (Mano's Basic Computer)
;
; Task : PROD = X * N by repeated addition (X = 5, N = 3 -> 15)
; CTR holds -N. ISZ adds 1 to it each pass and skips the BUN
; once it reaches 0, which ends the loop.
; ==============================================================
LOOP:   LDA PROD        ; AC <- running product
        ADD X           ; AC <- AC + X
        STA PROD        ; save it back
        ISZ CTR         ; CTR <- CTR + 1, skip next if CTR = 0
        BUN LOOP        ; not finished: go round again
        LDA PROD        ; AC <- final product
        HLT
X:      .data 1 5       ; multiplicand
CTR:    .data 1 -3      ; -N
PROD:   .data 1 0       ; product
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | 2009 | LOOP | LDA PROD |
| 1 | 001 | 1007 |  | ADD X |
| 2 | 002 | 3009 |  | STA PROD |
| 3 | 003 | 6008 |  | ISZ CTR |
| 4 | 004 | 4000 |  | BUN LOOP |
| 5 | 005 | 2009 |  | LDA PROD |
| 6 | 006 | 7001 |  | HLT |
| 7 | 007 | 0005 | X | .data 1 5 |
| 8 | 008 | FFFD | CTR | .data 1 -3 |
| 9 | 009 | 0000 | PROD | .data 1 0 |

## Procedure

1. Open `BasicComputer.cpu` and `P06_MEMORY_REFERENCE.a`.
2. Press **Ctrl+2**, then **Ctrl+D** to enter debug mode.
3. Set the Registers *Data* box to **Unsigned Dec** and the RAM *Data* box to **Hex**.
4. Click **Step by Instr** once per instruction (16 times) and record AC, DR, E, PC, AR and IR after each step.
5. When `HLT` runs the console shows "EXECUTION HALTED NORMALLY". **Start Over** repeats the trace.

<img width="1918" height="1128" alt="image" src="https://github.com/user-attachments/assets/bf2250a4-dcae-4f58-b033-72c537fe891e" />


<img width="1918" height="1125" alt="image" src="https://github.com/user-attachments/assets/906020f1-91ed-4aff-a0fc-5695c7ebff5c" />


<img width="1918" height="1103" alt="image" src="https://github.com/user-attachments/assets/1e538ccb-fb8c-494a-aba1-58fa10e9cb7e" />


<img width="1918" height="1127" alt="image" src="https://github.com/user-attachments/assets/76893c69-3b0d-40e0-82c6-8fd05955512c" />


<img width="1918" height="1127" alt="image" src="https://github.com/user-attachments/assets/4bf43b7b-6cf7-467d-b9a5-aacfa2fef56e" />


## Observations

Register contents (decimal) after each instruction:

| Step | PC before | Instruction | IR (hex) | AC | DR | E | PC | AR | IR (dec) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 0 | LDA PROD | 2009 | 0 | 0 | 0 | 1 | 9 | 8201 |
| 2 | 1 | ADD X | 1007 | 5 | 5 | 0 | 2 | 7 | 4103 |
| 3 | 2 | STA PROD | 3009 | 5 | 5 | 0 | 3 | 9 | 12297 |
| 4 | 3 | ISZ CTR | 6008 | 5 | 65534 (-2) | 0 | 4 | 8 | 24584 |
| 5 | 4 | BUN LOOP | 4000 | 5 | 65534 (-2) | 0 | 0 | 0 | 16384 |
| 6 | 0 | LDA PROD | 2009 | 5 | 5 | 0 | 1 | 9 | 8201 |
| 7 | 1 | ADD X | 1007 | 10 | 5 | 0 | 2 | 7 | 4103 |
| 8 | 2 | STA PROD | 3009 | 10 | 5 | 0 | 3 | 9 | 12297 |
| 9 | 3 | ISZ CTR | 6008 | 10 | 65535 (-1) | 0 | 4 | 8 | 24584 |
| 10 | 4 | BUN LOOP | 4000 | 10 | 65535 (-1) | 0 | 0 | 0 | 16384 |
| 11 | 0 | LDA PROD | 2009 | 10 | 10 | 0 | 1 | 9 | 8201 |
| 12 | 1 | ADD X | 1007 | 15 | 5 | 0 | 2 | 7 | 4103 |
| 13 | 2 | STA PROD | 3009 | 15 | 5 | 0 | 3 | 9 | 12297 |
| 14 | 3 | ISZ CTR | 6008 | 15 | 0 | 0 | 5 | 8 | 24584 |
| 15 | 5 | LDA PROD | 2009 | 15 | 15 | 0 | 6 | 9 | 8201 |
| 16 | 6 | HLT | 7001 | 15 | 15 | 0 | 7 | 1 | 28673 |

Values in brackets are the signed reading of the 16-bit number. Final memory: X = 5, CTR = 0, PROD = 15.

## Result

`LDA`, `ADD` and `STA` computed the running product, `ISZ` counted the passes and skipped the branch when the counter reached zero, and `BUN` formed the loop. Final **AC = PROD = 15**.

## Viva questions

**Q1. What is a memory-reference instruction?**
One whose operand is in memory; its 12-bit address field gives the location (opcodes 0–6 here).

**Q2. Why is CTR initialised to −3 and not 3?**
`ISZ` can only count upwards and test for zero, so counting up from −N reaches zero after N increments.

**Q3. What is the machine code of `ISZ CTR` with CTR at address 8?**
`6008` hex.

**Q4. What happens if CTR starts at 0?**
The first `ISZ` makes it 1, so the loop would run about 65536 times before CTR wraps back to 0.

**Q5. How does `ISZ` differ from `INC`?**
`ISZ` increments a *memory word* and may skip; `INC` increments AC and never skips.

---

# Practical 7: Register-reference Instructions: CLA, CMA, CME, HLT

**Aim:** To simulate `CLA`, `CMA`, `CME` and `HLT` and determine AC, E, PC, AR and IR (decimal) after each instruction.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

Register-reference instructions have the code 7xxx (opcode 111 with I = 0). The low 12 bits select one operation on AC or E, and no memory is accessed. Because the fetch routine always performs `AR ← IR(0–11)`, AR ends up holding the low 12 bits of the instruction code (800 hex = 2048 for `CLA`).

| Symbol | Code | Bit set in IR(0–11) | Operation |
| :--- | :--- | :--- | :--- |
| CLA | 7800 | B11 | AC ← 0 |
| CMA | 7200 | B9 | AC ← AC′ |
| CME | 7100 | B8 | E ← E′ |
| HLT | 7001 | B0 | S ← 1 (halt) |

An `LDA NUM` (AC ← 25) is placed first so the effect of `CLA` can be seen.

## Program – `programs/P07_REGISTER_REF_CLA_CMA_CME_HLT.a`

```
; ==============================================================
; Practical 7 : register-reference instructions CLA, CMA, CME, HLT
; Machine     : BasicComputer.cpu (Mano's Basic Computer)
; Step through in debug mode and note AC, E, PC, AR, IR (decimal).
; ==============================================================
        LDA NUM         ; AC <- 25, so CLA has something to clear
        CLA             ; 7800 : AC <- 0
        CMA             ; 7200 : AC <- AC'   (0000 -> FFFF)
        CME             ; 7100 : E <- E'     (0 -> 1)
        HLT             ; 7001 : halt
NUM:    .data 1 25
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | 2005 |  | LDA NUM |
| 1 | 001 | 7800 |  | CLA |
| 2 | 002 | 7200 |  | CMA |
| 3 | 003 | 7100 |  | CME |
| 4 | 004 | 7001 |  | HLT |
| 5 | 005 | 0019 | NUM | .data 1 25 |

## Procedure

1. Open `BasicComputer.cpu` and the program; press **Ctrl+2**, then **Ctrl+D**.
2. Set the Registers *Data* box to **Unsigned Dec** and the RAM *Data* box to **Hex**.
3. Click **Step by Instr** five times, recording AC, E, PC, AR and IR after each click.

<img width="1918" height="1132" alt="image" src="https://github.com/user-attachments/assets/b300e905-551d-46f1-92bc-17d75dcab80e" />


<img width="1918" height="1112" alt="image" src="https://github.com/user-attachments/assets/cb33eb29-da34-450a-8b12-d1bc942d6c3f" />


<img width="1918" height="1122" alt="image" src="https://github.com/user-attachments/assets/651af486-3ff8-4b90-8767-9561085409b9" />


<img width="1918" height="1125" alt="image" src="https://github.com/user-attachments/assets/1ddafe7e-53e1-4d3c-8c71-76e0842492c1" />

<img width="1918" height="1130" alt="image" src="https://github.com/user-attachments/assets/42c3ab6d-ca9e-43c8-92bb-4a757603324f" />


<img width="1918" height="1122" alt="image" src="https://github.com/user-attachments/assets/5a067433-5481-47e7-ae87-c70beec22d45" />


## Observations

Register contents (decimal) after each instruction:

| Step | PC before | Instruction | IR (hex) | AC | E | PC | AR | IR (dec) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 0 | LDA NUM | 2005 | 25 | 0 | 1 | 5 | 8197 |
| 2 | 1 | CLA | 7800 | 0 | 0 | 2 | 2048 | 30720 |
| 3 | 2 | CMA | 7200 | 65535 (-1) | 0 | 3 | 512 | 29184 |
| 4 | 3 | CME | 7100 | 65535 (-1) | 1 | 4 | 256 | 28928 |
| 5 | 4 | HLT | 7001 | 65535 (-1) | 1 | 5 | 1 | 28673 |

Final register contents after `HLT`:

| Register | Decimal | Hex |
| :--- | :--- | :--- |
| AC | 65535 (-1) | FFFF |
| E | 1 | 1 |
| PC | 5 | 005 |
| AR | 1 | 001 |
| IR | 28673 | 7001 |

AC is 65535 in *Unsigned Dec* and −1 in *Dec*; both are `FFFF`. PC = 5 because it holds the address after the `HLT` at address 4.

## Result

After execution: **AC = 65535 (−1), E = 1, PC = 5, AR = 1, IR = 28673.**

## Viva questions

**Q1. How does the control unit recognise a register-reference instruction?**
IR(14–12) = 111 and I = 0 (code 7xxx).

**Q2. Why is AR = 1 after `HLT`?**
Fetch does `AR ← IR(0–11)`, and the low 12 bits of `7001` are 001.

**Q3. Difference between `CLA` and `CLE`?**
`CLA` clears the 16-bit accumulator; `CLE` clears the 1-bit E flip-flop.

**Q4. Write two instructions that load −1 into AC.**
`CLA` followed by `CMA`.

---

# Practical 8: Register-reference Instructions: INC, SPA, SNA, SZE

**Aim:** To simulate `INC`, `SPA`, `SNA` and `SZE` and determine AC, E, PC, AR and IR (decimal) after each instruction.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

| Symbol | Code | Operation |
| :--- | :--- | :--- |
| INC | 7020 | AC ← AC + 1 |
| SPA | 7010 | if AC(15) = 0 (AC positive or zero) then PC ← PC + 1 |
| SNA | 7008 | if AC(15) = 1 (AC negative) then PC ← PC + 1 |
| SZE | 7002 | if E = 0 then PC ← PC + 1 |

A skip instruction increments PC one extra time when its condition is true, so the next instruction is not executed. In this program every skip is followed by a `HLT` "trap": the last `HLT` is reached only if every skip works. AC starts at −2 so that both a negative and a non-negative value are tested.

## Program – `programs/P08_REGISTER_REF_INC_SPA_SNA_SZE.a`

```
; ==============================================================
; Practical 8 : register-reference instructions INC, SPA, SNA, SZE
; Machine     : BasicComputer.cpu (Mano's Basic Computer)
; A true skip adds 1 to PC, so the next instruction is not run.
; Each HLT below is a trap: the last HLT is reached only if
; every skip behaved as expected.
; ==============================================================
        LDA NUM         ; AC <- -2
        INC             ; AC <- -1
        SNA             ; AC negative -> skip next
        HLT             ; (skipped)
        INC             ; AC <- 0
        SPA             ; sign bit 0 -> skip next
        HLT             ; (skipped)
        SZE             ; E = 0 -> skip next
        HLT             ; (skipped)
        INC             ; AC <- 1
        HLT             ; final halt
NUM:    .data 1 -2
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | 200B |  | LDA NUM |
| 1 | 001 | 7020 |  | INC |
| 2 | 002 | 7008 |  | SNA |
| 3 | 003 | 7001 |  | HLT |
| 4 | 004 | 7020 |  | INC |
| 5 | 005 | 7010 |  | SPA |
| 6 | 006 | 7001 |  | HLT |
| 7 | 007 | 7002 |  | SZE |
| 8 | 008 | 7001 |  | HLT |
| 9 | 009 | 7020 |  | INC |
| 10 | 00A | 7001 |  | HLT |
| 11 | 00B | FFFE | NUM | .data 1 -2 |

## Procedure

1. Open `BasicComputer.cpu` and the program; press **Ctrl+2**, then **Ctrl+D**.
2. Set the Registers *Data* box to **Unsigned Dec** and the RAM *Data* box to **Hex**.
3. Click **Step by Instr** eight times, recording AC, E, PC, AR and IR after each click.

<img width="1918" height="1140" alt="image" src="https://github.com/user-attachments/assets/745619c9-196c-4820-a558-e4b75f55ead3" />


<img width="1918" height="1113" alt="image" src="https://github.com/user-attachments/assets/81b772b6-2bb9-48fa-b92a-e82db2b8ecc0" />

<img width="1918" height="1122" alt="image" src="https://github.com/user-attachments/assets/d9b77c81-ca39-45fd-af24-c99a12fae9d5" />


<img width="1918" height="1123" alt="image" src="https://github.com/user-attachments/assets/f9115b12-18b8-4c0e-818a-2e227933624f" />


<img width="1918" height="1102" alt="image" src="https://github.com/user-attachments/assets/504a25e2-9b7c-449d-8a55-4c597c85a647" />


<img width="1918" height="1115" alt="image" src="https://github.com/user-attachments/assets/779522c9-ba01-43c0-8200-0ad504da33fe" />


<img width="1917" height="1121" alt="image" src="https://github.com/user-attachments/assets/705c3c6a-454b-4756-a797-50ea65877f8a" />


<img width="1918" height="1117" alt="image" src="https://github.com/user-attachments/assets/30414d66-b847-4e51-90ee-da28cf0295db" />


<img width="1918" height="1120" alt="image" src="https://github.com/user-attachments/assets/47990b4a-e1c8-4744-bff0-26c7a19bc2cb" />


## Observations

Register contents (decimal) after each instruction:

| Step | PC before | Instruction | IR (hex) | AC | E | PC | AR | IR (dec) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 0 | LDA NUM | 200B | 65534 (-2) | 0 | 1 | 11 | 8203 |
| 2 | 1 | INC | 7020 | 65535 (-1) | 0 | 2 | 32 | 28704 |
| 3 | 2 | SNA | 7008 | 65535 (-1) | 0 | 4 | 8 | 28680 |
| 4 | 4 | INC | 7020 | 0 | 0 | 5 | 32 | 28704 |
| 5 | 5 | SPA | 7010 | 0 | 0 | 7 | 16 | 28688 |
| 6 | 7 | SZE | 7002 | 0 | 0 | 9 | 2 | 28674 |
| 7 | 9 | INC | 7020 | 1 | 0 | 10 | 32 | 28704 |
| 8 | 10 | HLT | 7001 | 1 | 0 | 11 | 1 | 28673 |

The instructions at addresses 3, 6 and 8 were never executed: the PC column goes 2 → 4, 5 → 7 and 7 → 9.

Final register contents after `HLT`:

| Register | Decimal | Hex |
| :--- | :--- | :--- |
| AC | 1 | 0001 |
| E | 0 | 0 |
| PC | 11 | 00B |
| AR | 1 | 001 |
| IR | 28673 | 7001 |

## Result

`INC`, `SPA`, `SNA` and `SZE` were simulated and every skip was verified. After execution: **AC = 1, E = 0, PC = 11, AR = 1, IR = 28673.**

## Viva questions

**Q1. What does "skip" mean in the Basic Computer?**
PC is incremented once more, so the next instruction in memory is not executed.

**Q2. Does `SPA` skip when AC = 0?**
Yes. `SPA` tests only the sign bit, and zero has a sign bit of 0.

**Q3. How do you branch to X when AC is negative?**
`SPA` followed by `BUN X`: if AC is non-negative the `BUN` is skipped, otherwise it executes.

**Q4. Difference between `SZA` and `SZE`?**
`SZA` skips if AC = 0; `SZE` skips if E = 0.

**Q5. Why is AR = 32 after `INC`?**
Fetch loads AR with the low 12 bits of `7020`, which is `020` hex = 32.

---

# Practical 9: Register-reference Instructions: CIR, CIL

**Aim:** To simulate `CIR` and `CIL` and determine AC, E, PC, AR and IR (decimal) after each instruction.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

`CIR` and `CIL` circulate (rotate) the 17-bit combination of E and AC by one position:

```
CIR (7080): E -> AC(15) -> AC(14) -> ... -> AC(0) -> E     rotate right
CIL (7040): E <- AC(15) <- AC(14) <- ... <- AC(0) <- E     rotate left
```

No bit is lost, so a `CIR` followed by a `CIL` restores AC and E. CPU Sim has no 17-bit rotate, so each is built with the 1-bit scratch register TMP: the bit leaving AC is saved in TMP, AC is shifted, the old E enters the vacated end, and TMP is copied to E.

```
               E   AC
start          0   0000 0000 0000 1001  = 9
CIR            1   0000 0000 0000 0100  = 4      (AC bit 0 went into E)
CIR            0   1000 0000 0000 0010  = 32770  (old E entered AC bit 15)
CIL            1   0000 0000 0000 0100  = 4
CIL            0   0000 0000 0000 1001  = 9      (original value restored)
```

## Program – `programs/P09_REGISTER_REF_CIR_CIL.a`

```
; ==============================================================
; Practical 9 : register-reference instructions CIR, CIL
; Machine     : BasicComputer.cpu (Mano's Basic Computer)
; CIR rotates E and AC right (E -> AC15, AC0 -> E)
; CIL rotates E and AC left  (AC15 -> E, E -> AC0)
; ==============================================================
        LDA NUM         ; AC = 0000 0000 0000 1001 (9), E = 0
        CIR             ; AC = 0000 0000 0000 0100 (4), E = 1
        CIR             ; AC = 1000 0000 0000 0010,     E = 0
        CIL             ; AC = 0000 0000 0000 0100 (4), E = 1
        CIL             ; AC = 0000 0000 0000 1001 (9), E = 0
        HLT
NUM:    .data 1 9
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | 2006 |  | LDA NUM |
| 1 | 001 | 7080 |  | CIR |
| 2 | 002 | 7080 |  | CIR |
| 3 | 003 | 7040 |  | CIL |
| 4 | 004 | 7040 |  | CIL |
| 5 | 005 | 7001 |  | HLT |
| 6 | 006 | 0009 | NUM | .data 1 9 |

## Procedure

1. Open `BasicComputer.cpu` and the program; press **Ctrl+2**, then **Ctrl+D**.
2. Set the Registers *Data* box to **Unsigned Dec** and the RAM *Data* box to **Hex**.
3. Click **Step by Instr** six times, recording AC, E, PC, AR and IR after each click.

<img width="1918" height="1113" alt="image" src="https://github.com/user-attachments/assets/520c2420-6d21-4939-8d87-264996d5e61c" />


<img width="1918" height="1121" alt="image" src="https://github.com/user-attachments/assets/0758d2c2-b20b-477d-8413-680ec8e4347b" />


<img width="1918" height="1117" alt="image" src="https://github.com/user-attachments/assets/405dfa37-94a8-4414-aaa0-63341e76de23" />


<img width="1918" height="1113" alt="image" src="https://github.com/user-attachments/assets/2604eed6-7359-4cbd-97f1-bd4696769813" />


<img width="1917" height="1122" alt="image" src="https://github.com/user-attachments/assets/381b95a4-3e67-409d-a124-c40e682e941b" />


<img width="1918" height="1107" alt="image" src="https://github.com/user-attachments/assets/4a6873e0-2e5b-411e-8af0-8ac38516ff56" />


<img width="1918" height="1108" alt="image" src="https://github.com/user-attachments/assets/973dbc4e-6059-4066-9dd8-e0444e0f6fa5" />


## Observations

Register contents (decimal) after each instruction:

| Step | PC before | Instruction | IR (hex) | AC | E | PC | AR | IR (dec) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 0 | LDA NUM | 2006 | 9 | 0 | 1 | 6 | 8198 |
| 2 | 1 | CIR | 7080 | 4 | 1 | 2 | 128 | 28800 |
| 3 | 2 | CIR | 7080 | 32770 (-32766) | 0 | 3 | 128 | 28800 |
| 4 | 3 | CIL | 7040 | 4 | 1 | 4 | 64 | 28736 |
| 5 | 4 | CIL | 7040 | 9 | 0 | 5 | 64 | 28736 |
| 6 | 5 | HLT | 7001 | 9 | 0 | 6 | 1 | 28673 |

Final register contents after `HLT`:

| Register | Decimal | Hex |
| :--- | :--- | :--- |
| AC | 9 | 0009 |
| E | 0 | 0 |
| PC | 6 | 006 |
| AR | 1 | 001 |
| IR | 28673 | 7001 |

## Result

Two right rotations followed by two left rotations restored AC = 9. After execution: **AC = 9, E = 0, PC = 6, AR = 1, IR = 28673.** The values after each individual instruction are in the table above.

## Viva questions

**Q1. How does a circular shift differ from a logical shift?**
A circular shift feeds the bit shifted out back in at the other end (through E); a logical shift inserts 0 and loses the bit.

**Q2. How many bits take part in `CIR` / `CIL`?**
17: the 16 bits of AC plus E.

**Q3. How can `CIL` double AC?**
Clear E first (`CLE`); then `CIL` shifts AC left with a 0 entering bit 0, and E receives the old sign bit.

**Q4. What is AR after `CIR`?**
128, the low 12 bits of `7080` hex.

**Q5. After how many `CIL` operations do AC and E return to their starting values?**
17.

---

# Practical 10: Sum of Integers until a Negative Number is Read

**Aim:** To write an assembly program that reads integers and adds them until a negative non-zero number is read, then outputs the sum (not including that last number).

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

This is a sentinel-controlled loop: a negative number marks the end of the data. After each `INP`, `SPA` skips the exit branch when AC ≥ 0. For a negative number the skip does not happen, so `BUN DONE` leaves the loop *before* the number is added. Zero counts as non-negative, so it is added (it does not change the sum).

```
LOOP: INP -> AC >= 0 ?  --yes--> SUM = SUM + AC --> back to LOOP
                         |
                         no
                         v
DONE: LDA SUM -> OUT -> HLT
```

## Program – `programs/P10_SUM_UNTIL_NEGATIVE.a`

```
; ==============================================================
; Practical 10 : add integers until a negative number is read,
;                then print the sum (the negative is NOT added)
; Machine      : BasicComputer.cpu (Mano's Basic Computer)
; ==============================================================
LOOP:   INP             ; AC <- next number
        SPA             ; AC >= 0 ? then skip the exit branch
        BUN DONE        ; negative: leave the loop
        ADD SUM         ; AC <- AC + SUM
        STA SUM         ; SUM <- AC
        BUN LOOP        ; read the next number
DONE:   LDA SUM         ; AC <- SUM
        OUT             ; print the sum
        HLT
SUM:    .data 1 0       ; running total
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | F800 | LOOP | INP |
| 1 | 001 | 7010 |  | SPA |
| 2 | 002 | 4006 |  | BUN DONE |
| 3 | 003 | 1009 |  | ADD SUM |
| 4 | 004 | 3009 |  | STA SUM |
| 5 | 005 | 4000 |  | BUN LOOP |
| 6 | 006 | 2009 | DONE | LDA SUM |
| 7 | 007 | F400 |  | OUT |
| 8 | 008 | 7001 |  | HLT |
| 9 | 009 | 0000 | SUM | .data 1 0 |

## Procedure

1. Open `BasicComputer.cpu` and the program; set the RAM *Data* box to **Hex**.
2. Press **Ctrl+2**, then **Ctrl+R**.
3. Enter `4`, `10`, `0`, `6` and finally `-3`, pressing Enter after each.
4. The console prints `Output: 20`.

<img width="1918" height="1127" alt="image" src="https://github.com/user-attachments/assets/03e9d638-90de-4684-bb34-8f1e7394fddc" />



<img width="1918" height="1137" alt="image" src="https://github.com/user-attachments/assets/1f8e5b04-7ba3-420f-a7d8-30cdfa74ab9f" />


## Observations

| Input(s) typed | Output(s) displayed |
| :--- | :--- |
| 4, 10, 0, 6, -3 | 20 |
| -5 | 0 |
| 7, 8, -1 | 15 |

For the inputs 4, 10, 0, 6, −3 the loop adds four numbers; SUM (address 9) holds `0014` hex = 20. If the first number is negative the sum is 0.

## Result

The program adds integers until a negative number is read and displays the sum excluding it: 4 + 10 + 0 + 6 = **20**.

## Viva questions

**Q1. Why is `SPA` used and not `SNA`?**
`SPA` skips the exit branch for non-negative numbers, so `BUN DONE` runs only for negative input. With `SNA` the branches would have to be arranged the other way round.

**Q2. What happens when 0 is entered?**
Zero is non-negative, so it is added and the loop continues.

**Q3. Why is the negative number not included in the sum?**
The test is made right after `INP`, before `ADD SUM`.

**Q4. What is a sentinel value?**
A special input (here, any negative number) that marks the end of the data.

---

# Practical 11: Sum of Integers until Zero is Read

**Aim:** To write an assembly program that reads integers and adds them until zero is read, then outputs the sum.

**Tool:** CPU Sim 4.0.11 (Java 8 with JavaFX)

## Theory

Here the sentinel is 0. `SZA` skips the next instruction when AC = 0. A skip can jump over only one instruction, so two branches are used: when AC ≠ 0 the `BUN ADDIT` executes and the number is added; when AC = 0 that branch is skipped and `BUN DONE` ends the loop. Negative numbers are added normally.

## Program – `programs/P11_SUM_UNTIL_ZERO.a`

```
; ==============================================================
; Practical 11 : add integers until 0 is read, then print the sum
; Machine      : BasicComputer.cpu (Mano's Basic Computer)
; SZA skips only one instruction, so two branches are needed:
;   AC != 0 -> no skip -> BUN ADDIT runs
;   AC  = 0 -> skip BUN ADDIT -> BUN DONE ends the loop
; ==============================================================
LOOP:   INP             ; AC <- next number
        SZA             ; AC = 0 ? skip the next line
        BUN ADDIT       ; non-zero: go and add it
        BUN DONE        ; zero: finish
ADDIT:  ADD SUM         ; AC <- AC + SUM
        STA SUM         ; SUM <- AC
        BUN LOOP        ; read the next number
DONE:   LDA SUM         ; AC <- SUM
        OUT             ; print the sum
        HLT
SUM:    .data 1 0       ; running total
```

### Assembled program (memory map)

| Addr (dec) | Addr (hex) | Code (hex) | Label | Instruction |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 000 | F800 | LOOP | INP |
| 1 | 001 | 7004 |  | SZA |
| 2 | 002 | 4004 |  | BUN ADDIT |
| 3 | 003 | 4007 |  | BUN DONE |
| 4 | 004 | 100A | ADDIT | ADD SUM |
| 5 | 005 | 300A |  | STA SUM |
| 6 | 006 | 4000 |  | BUN LOOP |
| 7 | 007 | 200A | DONE | LDA SUM |
| 8 | 008 | F400 |  | OUT |
| 9 | 009 | 7001 |  | HLT |
| 10 | 00A | 0000 | SUM | .data 1 0 |

## Procedure

1. Open `BasicComputer.cpu` and the program; set the RAM *Data* box to **Hex**.
2. Press **Ctrl+2**, then **Ctrl+R**.
3. Enter `8`, `12`, `-5` and `0`, pressing Enter after each.
4. The console prints `Output: 15`.

<img width="1918" height="1133" alt="image" src="https://github.com/user-attachments/assets/8140ced8-e60f-4b64-84c9-b7c53f20e98c" />


<img width="1918" height="1131" alt="image" src="https://github.com/user-attachments/assets/3bb975d4-f37e-4e7c-9227-a4266c3c9101" />


## Observations

| Input(s) typed | Output(s) displayed |
| :--- | :--- |
| 8, 12, -5, 0 | 15 |
| 0 | 0 |
| 100, 200, 300, 0 | 600 |

For 8, 12, −5, 0 the final AC and SUM are 15. **E = 1** in the final register view, because adding −5 (`FFFB`) to 20 carries out of bit 15; the 16-bit result `000F` is still correct. (In the *Dec* view a 1-bit register holding 1 is shown as −1.)

## Result

The program adds integers until 0 is read and displays the sum: 8 + 12 + (−5) = **15**.


# Appendix: Quick reference

| Symbol | Hex | Operation |
| :--- | :--- | :--- |
| AND | 0xxx | DR ← M[AR]; AC ← AC ∧ DR |
| ADD | 1xxx | DR ← M[AR]; AC ← AC + DR, E ← Cout |
| LDA | 2xxx | DR ← M[AR]; AC ← DR |
| STA | 3xxx | M[AR] ← AC |
| BUN | 4xxx | PC ← AR |
| ISZ | 6xxx | DR ← M[AR]; DR ← DR + 1; M[AR] ← DR; if DR = 0 then PC ← PC + 1 |
| CLA | 7800 | AC ← 0 |
| CLE | 7400 | E ← 0 |
| CMA | 7200 | AC ← AC′ |
| CME | 7100 | E ← E′ |
| CIR | 7080 | rotate E and AC right |
| CIL | 7040 | rotate E and AC left |
| INC | 7020 | AC ← AC + 1 |
| SPA | 7010 | if AC(15) = 0 then PC ← PC + 1 |
| SNA | 7008 | if AC(15) = 1 then PC ← PC + 1 |
| SZA | 7004 | if AC = 0 then PC ← PC + 1 |
| SZE | 7002 | if E = 0 then PC ← PC + 1 |
| HLT | 7001 | S ← 1 (halt) |
| INP | F800 | AC ← integer typed by the user |
| OUT | F400 | display AC as an integer |

**Common problems**

| Symptom | Cause and fix |
| :--- | :--- |
| `Undefined symbol` / label error | A label is misspelt or has no colon. Labels are case-sensitive. |
| `Unknown instruction` | The loaded machine is not `BasicComputer.cpu`, or the mnemonic is not in capitals. |
| Console stays yellow | The program is waiting at an `INP`: type a number and press Enter. |
| Program never ends | A loop has no exit. Use *Execute → Stop* (Ctrl+.). |
| Warning when entering debug mode | Select **PC** as the program counter in *Execute → Options…*. |
| RAM shows little boxes | Set the RAM pane's *Data* box to Hex or Dec. |
| CPU Sim will not start | Java 8 **with JavaFX** is required (e.g. Azul Zulu JDK FX 8 or BellSoft Liberica JDK 8 Full); newer Java versions ship without JavaFX. |

Number bases in the Registers pane: **Dec** shows each register as a signed two's complement number of its own width (a 1-bit register holding 1 appears as −1); **Unsigned Dec** shows the plain binary value. Use *Unsigned Dec* for register traces and *Dec* for arithmetic results.
