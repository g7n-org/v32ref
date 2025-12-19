# v32ref: Vircon32 assembly language reference

An   information-dense,  quick-reference   markdown-formatted  guide   to
Vircon32  system  details and  assembly  language  mneumonics to  aid  in
development.

Information  in  this   document  is  based  on   the  official  Vircon32
Specification documents,  and the Vircon32  source code. Most  images are
from the specification documents.

This document is based on  Vircon32 DevTools **v25.1.19** or later; older
versions will contain inconsistencies.

## table of contents

  * [System Quick Reference](#system-quick-reference)
  * [CPU Error Codes](#cpu-error-codes)
  * [Assembler Data Directives](#assember-data-directives)
  * [Vircon32 Instruction Set Category Overview](#vircon32-instruction-set-category-overview)
  * [Vircon32 Instruction Set Detailed View](#vircon32-instruction-set-detailed-view)
  * [Vircon32 Registers](#vircon32-registers)
    * [General Purpose Registers](#general-purpose-registers)
    * [Internal Registers](#internal-registers)
    * [Stack Operations](#stack-operations)
    * [String Operations](#string-operations)
  * [Control Flags](#control-flags)
  * [Control Signals](#control-signals)
  * [Vircon32 Memory Map](#vircon32-memory-map)
  * [IOPorts](#ioports)
    * [Vircon32 IOPorts Layout](#vircon32-ioports-layout)
    * [TIME](#time)
    * [RNG](#rng)
    * [GPU](#gpu)
      * [GPU Commands](#gpu-commands)
      * [GPU Active Blending Port Commands](#gpu-active-blending-port-commands)
    * [SPU](#spu)
      * [SPU Commands](#spu-commands)
      * [SPU Channel States](#spu-channel-states)
    * [INPUT](#input)
    * [CARTRIDGE](#cartridge)
    * [MEMCARD](#memcard)
  * [Vircon32 instructions](#vircon32-instructions)
    * [Instruction Format](#instruction-format)
  * [Vircon32 ROM specification](#vircon32-rom-specification)

## system quick reference

| Attribute                   | Value                             |
| --------------------------- | --------------------------------- |
| VirconVersion               | 1                                 |
| VirconRevision              | 0                                 |
| FramesPerSecond             | 60                                |
| CyclesPerSecond             | 15,000,000                        |
| CyclesPerFrame              | CyclesPerSecond / FramesPerSecond |
| ScreenWidth                 | 640                               |
| ScreenHeight                | 360                               |
| GPUTextureSize              | 1024x1024                         |
| GPUMaximumCartridgeTextures | 256                               |
| GPURegionsPerTexture        | 4096                              |
| GPUPixelCapacityPerFrame    | 9 * ScreenPixels                  |
| GPUClearScreenPenalty       | -0.50f                            |
| GPUScalingPenalty           | +0.15f                            |
| GPURotationPenalty          | +0.25f                            |
| SPUMaximumCartridgeSounds   | 1024                              |
| SPUMaximumCartridgeSamples  | 1024 * 1024 * 256                 |
| SPUMaximumBiosSamples       | 1024 * 1024 * 1                   |
| SPUSoundChannels            | 16                                |
| SPUSamplingRate             | 44,100                            |
| SPUSamplesPerFrame          | SPUSamplingRate / FramesPerSecond |
| MaximumCartridgeProgramROM  | 1024 * 1024 * 128                 |
| MaximumBiosProgramROM       | 1024 * 1024 * 1                   |
| RAMSize                     | 1024 * 1024 * 4                   |
| MemoryCardSize              | 1024 * 256                        |
| MemoryBusSlaves             | 4                                 |
| ControlBusSlaves            | 8                                 |
| GamepadPorts                | 4                                 |

## CPU Error Codes

Runtime errors that Vircon32 generates:

| Value | Type               | Description                             |
| ----- | ------------------ | --------------------------------------- |
| 0     | InvalidMemoryRead  | read from invalid memory location       |
| 1     | InvalidMemoryWrite | write to invalid memory location        |
| 2     | InvalidPortRead    | invalid read from write-only port       |
| 3     | InvalidPortWrite   | invalid write to read-only port         |
| 4     | StackOverflow      | push would exceed lower bound of memory |
| 5     | StackUnderflow     | pop would exceed upper bound of memory  |
| 6     | DivisionError      | division by zero                        |
| 7     | ArcCosineError     | undefined value                         |
| 8     | ArcTangent2Error   | undefined value                         |
| 9     | LogarithmError     | undefined value                         |
| 10    | PowerError         | undefined value                         |

## Assembler Data Directives

| keyword  | description                  |
| -------- | ---------------------------- |
| integer  | specify one or more integers |
| float    | specify one or more floats   |
| string   | specify string sequence(s?)  |
| pointer  | specify pointer(s?)          |
| datafile | specify datafile(s?)         |

Use commas to separate values (create "array" of values)

## Vircon32 Instruction Set Category Overview

| branch/control | compare     | data          | logic         | arithmetic    | float math      |
| -------------- | ----------- | ------------- | ------------- | ------------- | --------------- |
| [HLT](#HLT)    | [IEQ](#IEQ) | [MOV](#MOV)   | [NOT](#NOT)   | [IADD](#IADD) | [FLR](#FLR)     |
| [WAIT](#WAIT)  | [INE](#INE) | [LEA](#LEA)   | [AND](#AND)   | [ISUB](#ISUB) | [CEIL](#CEIL)   |
| [JMP](#JMP)    | [IGT](#IGT) | [PUSH](#PUSH) | [OR](#OR)     | [IMUL](#IMUL) | [ROUND](#ROUND) |
| [CALL](#CALL)  | [IGE](#IGE) | [POP](#POP)   | [XOR](#XOR)   | [IDIV](#IDIV) | [SIN](#SIN)     |
| [RET](#RET)    | [ILT](#ILT) | [IN](#IN)     | [BNOT](#BNOT) | [IMOD](#IMOD) | [ACOS](#ACOS)   |
| [JT](#JT)      | [ILE](#ILE) | [OUT](#OUT)   | [SHL](#SHL)   | [ISGN](#ISGN) | [ATAN2](#ATAN2) |
| [JF](#JF)      | [FEQ](#FEQ) | [MOVS](#MOVS) |               | [IMIN](#IMIN) | [LOG](#LOG)     |
|                | [FNE](#FNE) | [SETS](#SETS) |               | [IMAX](#IMAX) | [POW](#POW)     |
|                | [FGT](#FGT) | [CMPS](#CMPS) |               | [IABS](#IABS) |                 |
|                | [FGE](#FGE) | [CIF](#CIF)   |               | [FADD](#FADD) |                 |
|                | [FLT](#FLT) | [CFI](#CFI)   |               | [FSUB](#FSUB) |                 |
|                | [FLE](#FLE) | [CIB](#CIB)   |               | [FMUL](#FMUL) |                 |
|                |             | [CFB](#CFB)   |               | [FDIV](#FDIV) |                 |
|                |             |               |               | [FMOD](#FMOD) |                 |
|                |             |               |               | [FSGN](#FSGN) |                 |
|                |             |               |               | [FMIN](#FMIN) |                 |
|                |             |               |               | [FMAX](#FMAX) |                 |
|                |             |               |               | [FABS](#FABS) |                 |

## Vircon32 Instruction Set Detailed View

There are 64 CPU opcodes, instructions  encode them in 6 bits. No invalid
opcodes can  exist. HLT is  opcode 0 for safety:  if an empty  or invalid
instruction is found, the CPU will stop execution.

| opcode | binary | mneumonic       | category   | operands | description                               |
| ------ | ------ | --------------- | ---------- | -------- | -----------                               |
| 0x00   | 000000 | [HLT](#HLT)     | control    | 0        | halt processing                           |
| 0x01   | 000001 | [WAIT](#WAIT)   | control    | 0        | pause processing, wait for next frame     |
| 0x02   | 000010 | [JMP](#JMP)     | branch     | 1        | unconditional jump to address             |
| 0x03   | 000011 | [CALL](#CALL)   | branch     | 1        | call subroutine                           |
| 0x04   | 000100 | [RET](#RET)     | branch     | 0        | return from subroutine                    |
| 0x05   | 000101 | [JT](#JT)       | branch     | 2        | jump if true (1)                          |
| 0x06   | 000110 | [JF](#JF)       | branch     | 2        | jump if false (0)                         |
| 0x07   | 000111 | [IEQ](#IEQ)     | compare    | 2        | integer equal                             |
| 0x08   | 001000 | [INE](#INE)     | compare    | 2        | integer not equal                         |
| 0x09   | 001001 | [IGT](#IGT)     | compare    | 2        | integer greater than                      |
| 0x0A   | 001010 | [IGE](#IGE)     | compare    | 2        | integer greater than or equal             |
| 0x0B   | 001011 | [ILT](#ILT)     | compare    | 2        | integer less than                         |
| 0x0C   | 001100 | [ILE](#ILE)     | compare    | 2        | integer less than or equal                |
| 0x0D   | 001101 | [FEQ](#FEQ)     | compare    | 2        | float equal                               |
| 0x0E   | 001110 | [FNE](#FNE)     | compare    | 2        | float not equal                           |
| 0x0F   | 001111 | [FGT](#FGT)     | compare    | 2        | float greater than                        |
| 0x10   | 010000 | [FGE](#FGE)     | compare    | 2        | float greater than or equal               |
| 0x11   | 010001 | [FLT](#FLT)     | compare    | 2        | float less than                           |
| 0x12   | 010010 | [FLE](#FLE)     | compare    | 2        | float less than or equal                  |
| 0x13   | 010011 | [MOV](#MOV)     | data       | 2        | copy data                                 |
| 0x14   | 010100 | [LEA](#LEA)     | data       | 2        | load effective address                    |
| 0x15   | 010101 | [PUSH](#PUSH)   | data       | 2        | push data to stack                        |
| 0x16   | 010110 | [POP](#POP)     | data       | 2        | pop data from stack                       |
| 0x17   | 010111 | [IN](#IN)       | data       | 2        | read data in from port                    |
| 0x18   | 011000 | [OUT](#OUT)     | data       | 2        | write data out to port                    |
| 0x19   | 011001 | [MOVS](#MOVS)   | data       | 0        | move string                               |
| 0x1A   | 011010 | [SETS](#SETS)   | data       | 0        | set string                                |
| 0x1B   | 011011 | [CMPS](#CMPS)   | data       | 1        | compare string                            |
| 0x1C   | 011100 | [CIF](#CIF)     | convert    | 1        | convert integer to float                  |
| 0x1D   | 011101 | [CFI](#CFI)     | convert    | 1        | convert float to integer                  |
| 0x1E   | 011110 | [CIB](#CIB)     | convert    | 1        | convert integer to boolean                |
| 0x1F   | 011111 | [CFB](#CFB)     | convert    | 1        | convert float to boolean                  |
| 0x20   | 100000 | [NOT](#NOT)     | logic      | 1        | perform bitwise NOT                       |
| 0x21   | 100001 | [AND](#AND)     | logic      | 2        | perform bitwise AND                       |
| 0x22   | 100010 | [OR](#OR)       | logic      | 2        | perform bitwise iOR                       |
| 0x23   | 100011 | [XOR](#XOR)     | logic      | 2        | perform bitwise XOR                       |
| 0x24   | 100100 | [BNOT](#BNOT)   | logic      | 1        | perform boolean NOT                       |
| 0x25   | 100101 | [SHL](#SHL)     | logic      | 2        | perform left shift                        |
| 0x26   | 100110 | [IADD](#IADD)   | arithmetic | 2        | perform integer addition                  |
| 0x27   | 100111 | [ISUB](#ISUB)   | arithmetic | 2        | perform integer subtraction               |
| 0x28   | 101000 | [IMUL](#IMUL)   | arithmetic | 2        | perform integer multiplication            |
| 0x29   | 101001 | [IDIV](#IDIV)   | arithmetic | 2        | perform integer division                  |
| 0x2A   | 101010 | [IMOD](#IMOD)   | arithmetic | 2        | perform integer modulus                   |
| 0x2B   | 101011 | [ISGN](#ISGN)   | arithmetic | 1        | perform integer sign toggle               |
| 0x2C   | 101100 | [IMIN](#IMIN)   | arithmetic | 2        | perform integer minimum                   |
| 0x2D   | 101101 | [IMAX](#IMAX)   | arithmetic | 2        | perform integer maximum                   |
| 0x2E   | 101110 | [IABS](#IABS)   | arithmetic | 1        | perform integer absolute value            |
| 0x2F   | 101111 | [FADD](#FADD)   | arithmetic | 2        | perform float addition                    |
| 0x30   | 110000 | [FSUB](#FSUB)   | arithmetic | 2        | perform float subtraction                 |
| 0x31   | 110001 | [FMUL](#FMUL)   | arithmetic | 2        | perform float multiplication              |
| 0x32   | 110010 | [FDIV](#FDIV)   | arithmetic | 2        | perform float division                    |
| 0x33   | 110011 | [FMOD](#FMOD)   | arithmetic | 2        | perform float modulus                     |
| 0x34   | 110100 | [FSGN](#FSGN)   | arithmetic | 1        | perform float sign toggle                 |
| 0x35   | 110101 | [FMIN](#FMIN)   | arithmetic | 2        | perform float minimum                     |
| 0x36   | 110110 | [FMAX](#FMAX)   | arithmetic | 2        | perform float maximum                     |
| 0x37   | 110111 | [FABS](#FABS)   | arithmetic | 1        | perform float absolute value              |
| 0x38   | 111000 | [FLR](#FLR)     | math       | 1        | perform float floor operation             |
| 0x39   | 111001 | [CEIL](#CEIL)   | math       | 1        | perform float ceiling operation           |
| 0x3A   | 111010 | [ROUND](#ROUND) | math       | 1        | perform float rounding operation          |
| 0x3B   | 111011 | [SIN](#SIN)     | math       | 1        | perform float sine operation              |
| 0x3C   | 111100 | [ACOS](#ACOS)   | math       | 1        | perform float arc cosine operation        |
| 0x3D   | 111101 | [ATAN2](#ATAN2) | math       | 2        | perform float arc tangent operation       |
| 0x3E   | 111110 | [LOG](#LOG)     | math       | 1        | perform float natural logarithm operation |
| 0x3F   | 111111 | [POW](#POW)     | math       | 2        | perform float power operation             |

## Vircon32 Registers

Vircon32 has 16 general purpose registers and 3 internal registers.

The general purpose  registers, which can be used  for either **integer**
or **floating point** operation (just: only one at any given moment).

Some registers  are used by various  instructions, so if used,  should be
avoided to prevent data hazards.

### General Purpose Registers

| Register | Binary | Alias | Description                  |
| -------- | ------ | ----- | ---------------------------- |
| R0       | 0000   |       |                              |
| R1       | 0001   |       |                              |
| R2       | 0010   |       |                              |
| R3       | 0011   |       |                              |
| R4       | 0100   |       |                              |
| R5       | 0101   |       |                              |
| R6       | 0110   |       |                              |
| R7       | 0111   |       |                              |
| R8       | 1000   |       |                              |
| R9       | 1001   |       |                              |
| R10      | 1010   |       |                              |
| R11      | 1011   | CR    | string: count register       |
| R12      | 1100   | SR    | string: source register      |
| R13      | 1101   | DR    | string: destination register |
| R14      | 1110   | BP    | stack: base pointer (base)   |
| R15      | 1111   | SP    | stack: stack pointer (top)   |

### Internal Registers

The Vircon32 internal registers are used  in the operation of the system,
and are  not directly  accessible to  the user  via programming.  Some of
these registers can be influenced based on actions taken.

| Register            | Description                                                        |
| ------------------- | ------------------------------------------------------------------ |
| InstructionPointer  | memory address from which the CPU will read the next instruction   |
| InstructionRegister | stores the last read instruction (the one currently executed)      |
| ImmediateValue      | if last read instruction sets the immediate flag, place value here |

The   **branch**  and   **control**   instructions   can  influence   the
**InstructionRegister**,  as redirecting  program flow  is the  result of
altering  the  register contents  (versus  just  proceeding to  the  next
instruction in sequence).

Any  instruction  that accepts  an  **Immediate**  value influences  this
internal **ImmediateValue** register.

### Stack Operations

The stack is  a convenient and often-used strategy on  the CPU. It relies
on the  tracking of  an otherwise  unused (or  carefully conflict-averse)
region of memory, typically located at the end of an addressable range of
RAM. There are two components: the base  of the stack, and the top of the
stack.

When you [PUSH](#PUSH) an item onto the stack, the stack pointer register
(**SP**) is decremented by one word, and the intended value stored at the
memory location stored therein.

When you [POP](#POP)  an item off the  stack, the value in  memory at the
current  location  stored  in  the stack  pointer  register  (**SP**)  is
obtained, then the stack pointer  register (**SP**) is incremented by one
word.

### String Operations

For a  small subset of  instructions, typically intended  for string-like
operations (in the  C sense: a string is an  array of individual values),
the CR/SR/DR registers can be  used to facilitate iterative operations on
data.

The three  instructions that utilize  this functionality on  Vircon32 are
[MOVS](#MOVS), [SETS](#SETS), and [CMPS](#CMPS).

## Control Flags

The CPU Control  Flags halt the CPU in  specific situations, interrupting
normal operations.

| Name     | Description                         |
| -------- | ----------------------------------- |
| HaltFlag | halt CPU until reset or power cycle |
| WaitFlag | halt CPU until next frame commences |

Flag values are 0 when reset, 1 when set.

## Control Signals

| signal | description                                                       |
| ------ | ----------------------------------------------------------------- |
| Reset  | Halt/Wait flags, registers reset; BP, SP, IP reset to defaults    |
| Frame  | Wait flag is reset                                                |
| Cycle  | if control flags set, do nothing; otherwise, do instruction cycle |

## Vircon32 Memory Map

| Name                            | Address    | Description                               |
| ------------------------------- | ---------- | ----------------------------------------- |
| RAMFirstAddress                 | 0x00000000 | read/write memory (16MB)                  |
| stack init address              | 0x003FFFFF | default location of SP (last RAM address) |
| BiosProgramROMFirstAddress      | 0x10000000 | Vircon32 BIOS                             |
| BIOS error handler address      | 0x10000000 | start of error handler logic              |
| BIOS program start address      | 0x10000004 | start of BIOS main logic                  |
| CartridgeProgramROMFirstAddress | 0x20000000 | Cartridge Data                            |
| MemoryCardRAMFirstAddress       | 0x30000000 | Memory Card Data                          |

# IOPorts

## Vircon32 IOPorts Layout

| Address | Vircon32 ID                 | Description                |
| ------- | --------------------------- | -------------------------- |
| 0x000   | [TIM_FirstPort](#TIME)      | time related functionality |
| 0x100   | [RNG_FirstPort](#RNG)       | random number generator    |
| 0x200   | [GPU_FirstPort](#GPU)       | graphics                   |
| 0x300   | [SPU_FirstPort](#SPU)       | sound processing           |
| 0x400   | [INP_FirstPort](#INPUT)     | input (game controllers)   |
| 0x500   | [CAR_FirstPort](#CARTRIDGE) | cartridge interface        |
| 0x600   | [MEM_FirstPort](#MEMCARD)   | memory card                |

## TIME

| Type | Port  | Name             | Description                  |
| ---- | ----- | ---------------- | ---------------------------- |
| IN   | 0x000 | TIM_CurrentDate  | retrieve current date        |
| IN   | 0x001 | TIM_CurrentTime  | retrieve current time        |
| IN   | 0x002 | TIM_FrameCounter | retrieve current frame count |
| IN   | 0x003 | TIM_CycleCounter | retrieve current cycle count |

### example: get current frame count

```
    IN R0,  TIM_FrameCounter    ; load current frame count into R0
```

## RNG

| Type  | Port  | Name             | Description                  |
| ----- | ----- | ---------------- | ---------------------------- |
| IN    | 0x100 | RNG_CurrentValue | obtain pseudorandom value    |
| OUT   | 0x100 | RNG_CurrentValue | seed random number generator |

## GPU

| Type  | Port  | Name                | Description                                    |
| ----- | ----- | ------------------- | ---------------------------------------------- |
| OUT   | 0x200 | GPU_Command         | perform GPU operation                          |
| IN    | 0x201 | GPU_RemainingPixels | ???                                            |
| IN    | 0x202 | GPU_ClearColor      | obtain current clear color                     |
| OUT   | 0x202 | GPU_ClearColor      | color to clear the screen with                 |
| IN    | 0x203 | GPU_MultiplyColor   | obtain current color multiplier                |
| OUT   | 0x203 | GPU_MultiplyColor   | color multiplier to draw sprites with          |
| IN    | 0x204 | GPU_ActiveBlending  | obtain current blending mode                   |
| OUT   | 0x204 | GPU_ActiveBlending  | blending method to draw sprites with           |
| IN    | 0x204 | GPU_SelectedTexture | obtain current selected texture                |
| OUT   | 0x204 | GPU_SelectedTexture | texture ID to select (-1 for BIOS)             |
| IN    | 0x205 | GPU_SelectedRegion  | obtain current selected region                 |
| OUT   | 0x205 | GPU_SelectedRegion  | region ID to select                            |
| IN    | 0x206 | GPU_DrawingPointX   | obtain X position to draw selected region      |
| OUT   | 0x206 | GPU_DrawingPointX   | set X position to draw selected region         |
| IN    | 0x207 | GPU_DrawingPointY   | obtain Y position to draw selected region      |
| OUT   | 0x207 | GPU_DrawingPointY   | set Y position to draw selected region         |
| IN    | 0x208 | GPU_DrawingScaleX   | obtain X scaling as a float                    |
| OUT   | 0x208 | GPU_DrawingScaleX   | sets X scaling with a float as input           |
| IN    | 0x209 | GPU_DrawingScaleY   | obtain Y scaling as a float                    |
| OUT   | 0x209 | GPU_DrawingScaleY   | sets Y scaling with a float as input           |
| IN    | 0x20A | GPU_DrawingAngle    | obtain the sprite rotation as a float          |
| OUT   | 0x20A | GPU_DrawingAngle    | sets the sprite rotation with a float as input |
| IN    | 0x20B | GPU_RegionMinX      | obtain Min X coordinate for region             |
| OUT   | 0x20B | GPU_RegionMinX      | set Min X coordinate for region                |
| IN    | 0x20C | GPU_RegionMinY      | obtain Min Y coordinate for region             |
| OUT   | 0x20C | GPU_RegionMinY      | set Min Y coordinate for region                |
| IN    | 0x20D | GPU_RegionMaxX      | obtain Max X coordinate for region             |
| OUT   | 0x20D | GPU_RegionMaxX      | set Max X coordinate for region                |
| IN    | 0x20E | GPU_RegionMaxY      | obtain Max Y coordinate for region             |
| OUT   | 0x20E | GPU_RegionMaxY      | set Max Y coordinate for region                |
| IN    | 0x20F | GPU_RegionHotspotX  | obtain region Hotspot X coordinate             |
| OUT   | 0x20F | GPU_RegionHotspotX  | set region Hotspot X coordinate                |
| IN    | 0x210 | GPU_RegionHotspotY  | obtain region Hotspot Y coordinate             |
| OUT   | 0x210 | GPU_RegionHotspotY  | set region Hotspot Y coordinate                |

### GPU Commands

Commands that can be issued to the GPU:

| Value | Name                            | Description                                       |
| ----- | ------------------------------- | ------------------------------------------------- |
| 0x10  | GPUCommand_ClearScreen          | clears the screen using current clear color       |
| 0x11  | GPUCommand_DrawRegion           | draws the selected region: rotation off, zoom off |
| 0x12  | GPUCommand_DrawRegionZoomed     | draws the selected region: rotation off, zoom on  |
| 0x13  | GPUCommand_DrawRegionRotated    | draws the selected region: rotation on , zoom off |
| 0x14  | GPUCommand_DrawRegionRotozoomed | draws the selected region: rotation on , zoom on  |

### GPU Active Blending Port Commands

Active blending:

| Value | Name                     | Description                                             |
| ----- | ------------------------ | ------------------------------------------------------- |
| 0x20  | GPUBlendingMode_Alpha    | default rendering, uses alpha channel as transparency   |
| 0x21  | GPUBlendingMode_Add      | colors are added (light effect): aka "linear dodge"     |
| 0x22  | GPUBlendingMode_Subtract | colors are subtracted (shadow effect): aka "difference" |

## SPU

| Type | Port  | Name                     | Description                                    |
| ---- | ----- | ------------------------ | ---------------------------------------------- |
| OUT  | 0x300 | SPU_Command              | perform SPU operation                          |
| ???  | 0x301 | SPU_GlobalVolume         | ???  |
| OUT  | 0x302 | SPU_SelectedSound        | ???  |
| OUT  | 0x303 | SPU_SelectedChannel      | ???  |
| ???  | 0x304 | SPU_SoundLength          | ???  |
| ???  | 0x305 | SPU_SoundPlayWithLoop    | ???  |
| ???  | 0x306 | SPU_SoundLoopStart       | ???  |
| ???  | 0x307 | SPU_SoundLoopEnd         | ???  |
| ???  | 0x308 | SPU_ChannelState         | ???  |
| ???  | 0x309 | SPU_ChannelAssignedSound | ???  |
| ???  | 0x30A | SPU_ChannelVolume        | ???  |
| ???  | 0x30B | SPU_ChannelSpeed         | ???  |
| ???  | 0x30C | SPU_ChannelLoopEnabled   | ???  |
| ???  | 0x30D | SPU_ChannelPosition      | ???  |

### SPU Commands

Commands for the SPU:

| Value | Name                            | Description                                         |
| ----- | ------------------------------- | --------------------------------------------------- |
| 0x30  | SPUCommand_PlaySelectedChannel  | if paused, resume; if playing, retriggered          |
| 0x31  | SPUCommand_PauseSelectedChannel | pause; no effect if the channel was not playing     |
| 0x32  | SPUCommand_StopSelectedChannel  | position is rewinded to sound start                 |
| 0x33  | SPUCommand_PauseAllChannels     | same as applying PauseChannel to all channels       |
| 0x34  | SPUCommand_ResumeAllChannels    | same as applying PlayChannel to all paused channels |
| 0x35  | SPUCommand_StopAllChannels      | same as applying StopChannel to all channels        |

### SPU Channel States

States of the sound channels:

| Value | Name                    | Description                                                     |
| ----- | ----------------------- | --------------------------------------------------------------- |
| 0x40  | SPUChannelState_Stopped | channel is not playing, and will begin new reproduction on play |
| 0x41  | SPUChannelState_Paused  | channel is paused, and will resume reproduction on play         |
| 0x42  | SPUChannelState_Playing | channel is currently playing, until its assigned sound ends     |

## INPUT

| Type | Port  | Name                   | Description                            |
| ---- | ----- | ---------------------- | -------------------------------------- |
| IN   | 0x400 | INP_SelectedGamepad    | read which gamepad is selected (0-3)   |
| OUT  | 0x400 | INP_SelectedGamepad    | select indicated gamepad (0-3)         |
| IN   | 0x401 | INP_GamepadConnected   | status of gamepad being connected      |
| IN   | 0x402 | INP_GamepadLeft        | Left button input                      |
| IN   | 0x403 | INP_GamepadRight       | Right button input                     |
| IN   | 0x404 | INP_GamepadUp          | Up button input                        |
| IN   | 0x405 | INP_GamepadDown        | Down button input                      |
| IN   | 0x406 | INP_GamepadButtonStart | Start button input (Enter on keyboard) |
| IN   | 0x407 | INP_GamepadButtonA     | A button input (X on keyboard)         |
| IN   | 0x408 | INP_GamepadButtonB     | B button input (Z on keyboard)         |
| IN   | 0x409 | INP_GamepadButtonX     | X key input (S on keyboard)            |
| IN   | 0x40A | INP_GamepadButtonY     | Y key input (A on keyboard)            |
| IN   | 0x40B | INP_GamepadButtonL     | L key input (Q on keyboard)            |
| IN   | 0x40C | INP_GamepadButtonR     | R key input (W on keyboard)            |

## CARTRIDGE

| Type | Port  | Name                 | Description                         |
| ---- | ----- | -------------------- | ----------------------------------- |
| IN   | 0x500 | CAR_Connected        | status of cartridge being connected |
| IN   | 0x501 | CAR_ProgramROMSize   | size of program ROM                 |
| IN   | 0x502 | CAR_NumberOfTextures | number of cartridge textures        |
| IN   | 0x503 | CAR_NumberOfSounds   | number of cartridge sounds          |

## MEMCARD

| Type | Port  | Name          | Description                           |
| ---- | ----- | ------------- | ------------------------------------- |
| IN?  | 0x600 | MEM_Connected | status of memory card being connected |

# Vircon32 instructions

## Instruction Format

![V32 instruction format](V32_instruction_format.jpg)

| bits  | description            |
| ----- | ---------------------- |
| 31-26 | opcode                 |
| 25    | immediate value flag   |
| 24-21 | destination register   |
| 20-17 | source register        |
| 16-14 | address mode (for MOV) |
| 13-0  | port number            |

If the **immediate value**  bit is set, an additional word  is read to be
used as an operand to the instruction.

## HLT

Halt CPU

### Description

HLT sets the CPU’s Halt flag. This will cause the CPU to stop execution
until the flag is cleared at the next console power on or reset.

Note that other components will keep functioning: for instance,  if the
SPU was playing music it will continue to do so.

### Variants and Actions

| Form      | Processing Action   |
| --------- | ------------------- |
| ```HLT``` | ```HaltFlag = 1;``` |

## WAIT

Pause execution until the next frame starts

### Description

WAIT activates  the CPU’s Wait flag.  This will cause the  CPU to pause
execution until the  flag is cleared when the timer  signals the start of
the next frame. Reset or power-on actions will also resume CPU execution,
since they also  cause a new frame  to begin.

When the  new frame begins, the  CPU will resume execution  following the
usual order,  i.e. processing the  instruction directly after  WAIT. Note
that other components will keep functioning

### Variants and Actions

| Form       | Processing Action   |
| ---------- | ------------------- |
| ```WAIT``` | ```WaitFlag = 1;``` |

## JMP

Unconditional jump.

### Description

**JMP**  performs  an unconditional  jump  to  the address  specified  by
its  operand. After  processing this  instruction the  CPU will  continue
execution at the new address.

This causes a forcible redirect of program flow to the indicated address;
this is done  by causing the **InstructionPointer**  internal register to
be modified. The  address is somewhere else in the  program logic, likely
identified by some set label.

### Variants and Actions

| Form                | Processing Action                     |
| ------------------- | ------------------------------------- |
| ```JMP Immediate``` | ```InstructionPointer = Immediate;``` |
| ```JMP SRCREG```    | ```InstructionPointer = SRCREG;```    |

## CALL

Call subroutine

### Description

CALL performs a subroutine call to the specified address. When processing
this  instruction,  the current  Instruction  Pointer  (that was  already
incremented) will be  saved to the top  of the stack and then  it will be
overwritten  with the  new address.  Execution will  continue at  the new
address.

### Variants and Actions

| Form                 | Processing Action                                                     |
| -------------------- | --------------------------------------------------------------------- |
| ```CALL Immediate``` | ```Stack.Push(InstructionPointer); InstructionPointer = Immediate;``` |
| ```CALL SRCREG```    | ```Stack.Push(InstructionPointer); InstructionPointer = SRCREG;```    |

04 Instruction RET (Return)
Structure and variants:
RET
Processing actions:
InstructionPointer = Stack.Pop( )

## RET

Return from subroutine

### Description

RET returns  from a  previously called  subroutine. When  processing this
instruction, the current Instruction Pointer will be overwritten with the
topmost  value  in  the  stack.  Execution will  then  continue  at  that
previously saved address.

### Variants and Actions

| Form      | Processing Action                        |
| --------- | ---------------------------------------- |
| ```RET``` | ```InstructionPointer  = Stack.Pop();``` |

## JT

Jump if  True: a conditional  jump typically used following  a comparison
instruction, should the  queried register contain a true  (1) value, jump
to indicated address.

### Description

JT performs a jump only if its  first operand is true, i.e. non zero when
taken  as an  integer.  In that  case  its  behavior is  the  same as  an
unconditional jump. Otherwise it has no effect.

### NOTE

For the purposes of comparisons and conditional jumps on Vircon32:

  * true is 1 (technically non-zero)
  * false is 0

### Variants and Actions

| Form                        | Processing Action                                     |
| --------------------------- | ----------------------------------------------------- |
| ```JT  DSTREG, Immediate``` | ```InstructionPointer = (DSTREG != 0) ? Immediate;``` |
| ```JT  DSTREG, SRCREG```    | ```InstructionPointer = (DSTREG != 0) ? SRCREG;```    |

## JF

Jump if False:  a conditional jump typically used  following a comparison
instruction, should the queried register  contain a false (0) value, jump
to indicated address.

### Description

JF performs  a jump only  if its first operand  is false, i.e.  zero when
taken  as an  integer.  In that  case  its  behavior is  the  same as  an
unconditional jump. Otherwise it has no effect.

### NOTE

For the purposes of comparisons and conditional jumps on Vircon32:

  * true is 1 (technically non-zero)
  * false is 0

### Variants and Actions

| Form                        | Processing Action                                     |
| --------------------------- | ----------------------------------------------------- |
| ```JF  DSTREG, Immediate``` | ```InstructionPointer = (DSTREG == 0) ? Immediate;``` |
| ```JF  DSTREG, SRCREG```    | ```InstructionPointer = (DSTREG == 0) ? SRCREG;```    |

## Comparisons

For the purposes of comparisons and conditional jumps on Vircon32:

  * true is 1 (technically non-zero)
  * false is 0

There are six relational operations (for integers and floats):

  * (xEQ) is equal to
  * (xNE) is not equal to
  * (xLT) is less than
  * (xLE) is than or equal to
  * (xGT) is greater than
  * (xGE) is greater than or equal to

## IEQ

Integer Compare Equality

### Description

IEQ takes  two operands interpreted as  integers, and checks if  they are
equal. It  will store the boolean  result in the first  operand, which is
always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```IEQ DSTREG, Immediate``` | ```DSTREG = (DSTREG == Immediate) ? 1 : 0;``` |
| ```IEQ DSTREG, SRCREG```    | ```DSTREG = (DSTREG == SRCREG) ? 1 : 0;```    |

## INE

Integer Not Equal

### Description

INE takes  two operands interpreted as  integers, and checks if  they are
different. It will  store the boolean result in the  first operand, which
is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```INE DSTREG, Immediate``` | ```DSTREG = (DSTREG != Immediate) ? 1 : 0;``` |
| ```INE DSTREG, SRCREG```    | ```DSTREG = (DSTREG != SRCREG) ? 1 : 0;```    |

## IGT

Integer  Greater Than

### Description

IGT takes two  operands interpreted as integers, and checks  if the first
one is greater than  the second. It will store the  boolean result in the
first operand, which is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```IGT DSTREG, Immediate``` | ```DSTREG = (DSTREG >  Immediate) ? 1 : 0;``` |
| ```IGT DSTREG, SRCREG```    | ```DSTREG = (DSTREG >  SRCREG) ? 1 : 0;```    |

## IGE

Integer Greater Than Or Equal

### Description

IGE takes two  operands interpreted as integers, and checks  if the first
one is greater or  equal to the second. It will  store the boolean result
in the first operand, which is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```IGE DSTREG, Immediate``` | ```DSTREG = (DSTREG >= Immediate) ? 1 : 0;``` |
| ```IGE DSTREG, SRCREG```    | ```DSTREG = (DSTREG >= SRCREG) ? 1 : 0;```    |

## ILT

Integer Less Than

### Description

ILT takes two  operands interpreted as integers, and checks  if the first
one is  less than  the second. It  will store the  boolean result  in the
first operand, which is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```ILT DSTREG, Immediate``` | ```DSTREG = (DSTREG <  Immediate) ? 1 : 0;``` |
| ```ILT DSTREG, SRCREG```    | ```DSTREG = (DSTREG <  SRCREG) ? 1 : 0;```    |

## ILE

Integer Less  Than Or Equal

### Description

ILE takes two  operands interpreted as integers, and checks  if the first
one is less or  equal to the second. It will store  the boolean result in
the first operand, which is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```ILE DSTREG, Immediate``` | ```DSTREG = (DSTREG <= Immediate) ? 1 : 0;``` |
| ```ILE DSTREG, SRCREG```    | ```DSTREG = (DSTREG <= SRCREG) ? 1 : 0;```    |

## FEQ

Float Compare Equality

### Description:

FEQ takes  two operands  interpreted as  floats, and  checks if  they are
equal. It  will store the boolean  result in the first  operand, which is
always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```FEQ DSTREG, Immediate``` | ```DSTREG = (DSTREG == Immediate) ? 1 : 0;``` |
| ```FEQ DSTREG, SRCREG```    | ```DSTREG = (DSTREG == SRCREG) ? 1 : 0;```    |

## FNE

Float Not Equal

### Description

FNE takes  two operands  interpreted as  floats, and  checks if  they are
different. It will  store the boolean result in the  first operand, which
is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```FNE DSTREG, Immediate``` | ```DSTREG = (DSTREG != Immediate) ? 1 : 0;``` |
| ```FNE DSTREG, SRCREG```    | ```DSTREG = (DSTREG != SRCREG) ? 1 : 0;```    |

## FGT

Float Greater Than

### Description

FGT takes two operands interpreted as floats, and checks if the first one
is greater than the second. It will store the boolean result in the first
operand, which is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```FGT DSTREG, Immediate``` | ```DSTREG = (DSTREG >  Immediate) ? 1 : 0;``` |
| ```FGT DSTREG, SRCREG```    | ```DSTREG = (DSTREG >  SRCREG) ? 1 : 0;```    |

## FGE

Float Greater Than or Equal

### Description

FGE takes two  operands interpreted as integers, and checks  if the first
one is greater or  equal to the second. It will  store the boolean result
in the first operand, which is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```FGE DSTREG, Immediate``` | ```DSTREG = (DSTREG >= Immediate) ? 1 : 0;``` |
| ```FGE DSTREG, SRCREG```    | ```DSTREG = (DSTREG >= SRCREG) ? 1 : 0;```    |

## FLT

Float Less Than

### Description

FLT takes two operands interpreted as floats, and checks if the first one
is less than  the second. It will  store the boolean result  in the first
operand, which is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```FLT DSTREG, Immediate``` | ```DSTREG = (DSTREG <  Immediate) ? 1 : 0;``` |
| ```FLT DSTREG, SRCREG```    | ```DSTREG = (DSTREG <  SRCREG) ? 1 : 0;```    |

## FLE

Float Less Than or Equal

### Description

FLE takes two operands interpreted as floats, and checks if the first one
is less or equal  to the second. It will store the  boolean result in the
first operand, which is always a register.

### Variants and Actions

| Form                        | Processing Action                             |
| --------------------------- | --------------------------------------------- |
| ```FLE DSTREG, Immediate``` | ```DSTREG = (DSTREG <= Immediate) ? 1 : 0;``` |
| ```FLE DSTREG, SRCREG```    | ```DSTREG = (DSTREG <= SRCREG) ? 1 : 0;```    |

## MOV

MOVE: general purpose data-copying instruction.

Indirect  processing  is  accomplished  with the  **```[  ]```**  (square
brackets)  surrounding  the  value  we wish  to  dereference  (we're  not
interested in  the direct  thing: the memory  address, but  indirectly in
what that memory address contains).

### Description

MOV copies  the value indicated in  its second (source) operand  into the
register or memory address indicated  by its first (destination) operand.
MOV  is the  most  complex instruction  to process  because  it needs  to
distinguish between 8 different addressing modes.

The  instruction  specifies   which  of  the  8  modes  to   use  in  its
“Addressing  mode” field,  being the  possible values  interpreted as
follows:

### Variants and Actions

| Type              | Binary | Form                           | Processing Action               |
| ----------------- | ------ | ------------------------------ | ------------------------------- |
| RegFromImm        | 000    | ```MOV DSTREG, Imm```          | ```DSTREG = Imm;```             |
| RegFromReg        | 001    | ```MOV DSTREG, SRCREG```       | ```DSTREG = SRCREG;```          |
| RegFromImmAddr    | 010    | ```MOV DSTREG, [Imm]```        | ```DSTREG = MEM[Imm];```        |
| RegFromRegAddr    | 011    | ```MOV DSTREG, [SRCREG]```     | ```DSTREG = MEM[SRCREG];```     |
| RegFromAddrOffset | 100    | ```MOV DSTREG, [SRCREG+Imm]``` | ```DSTREG = MEM[SRCREG+Imm];``` |
| ImmAddrFromReg    | 101    | ```MOV [Imm], SRCREG```        | ```MEM[Imm] = SRCREG;```        |
| RegAddrFromReg    | 110    | ```MOV [DSTREG], SRCREG```     | ```MEM[DSTREG] = SRCREG;```     |
| AddrOffsetFromReg | 111    | ```MOV [DSTREG+Imm], SRCREG``` | ```MEM[DSTREG+Imm] = SRCREG;``` |

| KEY          | Description                          |
| ------------ | ------------------------------------ |
| Imm          | Immediate Value: ```0xAF```          |
| Reg          | Register: ```R7```                   |
| ImmAddr      | ImmediateAddress: ```[0x00004FFF]``` |
| RegAddr      | RegisterAddress: ```[R3]```          |
| AddrOffset   | AddressOffset: ```[R1+73]```         |
| ```MEM[]```  | Memory access                        |
| ```DSTREG``` | Destination Register (first operand) |
| ```SRCREG``` | Source Register (second operand)     |

## LEA

Load Effective Address of a memory position.

### Description

LEA takes a memory address as second operand. It stores that address (not
its contents) into  the register given as first operand.  The most useful
case is when the address is given in the form pointer + offset, since the
addition is automatically performed.

### Variants and Actions

| Form                                 | Processing Action                  |
| ------------------------------------ | ---------------------------------- |
| ```LEA DSTREG, [SRCREG]```           | ```DSTREG = SRCREG;```             |
| ```LEA DSTREG, [SRCREG+Immediate]``` | ```DSTREG = SRCREG + Immediate;``` |

## PUSH

Store register contents to top of stack

### Description

PUSH uses the CPU hardware stack to  add the value contained in the given
register at the top  of the stack. When you PUSH a  value onto the stack,
the STACK POINTER (SP) is adjusted  downward by one address offset (stack
grows down).

### Variants and Actions

| Form              | Processing Action         |
| ----------------- | ------------------------- |
| ```PUSH SRCREG``` | ```Stack.Push(SRCREG);``` |

## POP

Load from top of stack to register

### Description

POP uses  the CPU hardware stack  to remove a  value from the top  of the
stack and write  it in the given  register. When you POP a  value off the
stack, the  STACK POINTER (SP) is  adjusted upward by one  address offset
(stack grows down, shrinks up).

### Variants and Actions

| Form             | Processing Action           |
| ---------------- | --------------------------- |
| ```POP DSTREG``` | ```DSTREG = Stack.Pop();``` |

## IN

Read from an I/O port

### Description

In uses  the control bus  to read  from an I/O  port in another  chip and
stores the  returned value in  the specified register. This  read request
may lead to side effects depending on the specified port.

### Variants and Actions

| Form                        | Processing Action                |
| --------------------------- | -------------------------------- |
| ```IN DSTREG, PortNumber``` | ```DSTREG = Port[PortNumber];``` |

## OUT

Write to an I/O port

### Description

OUT uses the control  bus to write the specified value to  an I/O port in
another chip.  This write request may  lead to side effects  depending on
the specified port.

### Variants and Actions

| Form                            | Processing Action                   |
| ------------------------------- | ----------------------------------- |
| ```OUT PortNumber, Immediate``` | ```Port[PortNumber] = Immediate;``` |
| ```OUT PortNumber, SRCREG```    | ```Port[PortNumber] = SRCREG;```    |

## MOVS

Copy string (HW memcpy)

### Description

MOVS copies  a value  from the memory  address pointed by  SR to  the one
pointed by  DR (as in  a supposed MOV [DR],  [SR]). It then  implements a
local loop  to repeat  itself until  the counter in  CR reaches  0, while
working on consecutive addresses.

Note that even when called with a value  of CR of zero or less, MOVS will
always perform the described loop at least once.

This instruction is the only way  in Vircon32 CPU to directly copy values
from 2 places in memory without going through a register.

### Variants and Actions

| Form       | Processing Action                            |
| ---------- | -------------------------------------------- |
| ```MOVS``` | see below                                    |

### Processing actions

```
do
{
    MEM[DR]  = MEM[SR];
    DR       = DR + 1; // DR: Destination Register (R13)
    SR       = SR + 1; // SR: Source Register (R12)
    CR       = CR - 1; // CR: Count Register (R11)
    IP       = IP - 1; // IP: InstructionPointer internal register
} while (CR > 0);
```

## SETS

Set string (HW memset)

### Description

SETS copies  the value in SR  to the address pointed  by DR (as in  a MOV
[DR], SR).  It then implements  a local loop  to repeat itself  until the
counter in CR reaches 0, while writing to consecutive addresses.

Note that even when called with a value  of CR of zero or less, SETS will
always perform the described loop at least once.

### Variants and Actions

| Form       | Processing Action                            |
| ---------- | -------------------------------------------- |
| ```SETS``` | see below                                    |

### Processing actions

```
do
{
    Memory[DR]  = SR;     // SR: Source Register (R12)
    DR          = DR + 1; // DR: Destination Register (R13)
    CR          = CR - 1; // CR: Count Register (R11)
    IP          = IP - 1; // IP: InstructionPointer internal register
} while (CR > 0);
```

## CMPS

Compare string (HW memcmp)

### Description

CMPS takes as  a reference the compares the value  in the address pointed
by DR and compares it with the one pointed by SR, by subtracting. It then
implements a local loop to repeat  itself until the counter in CR reaches
0, while  reading consecutive  addresses. The  comparison result  will be
stored in the  specified register, and will be zero  when equal, positive
when some value at [DR] was greater, and negative when some value in [SR]
was greater.

Note that even when called with a value  of CR of zero or less, CMPS will
always perform the described loop at least once.

### Variants and Actions

| Form           | Processing Action                            |
| -------------- | -------------------------------------------- |
| ```CMPS REG``` | see below                                    |

### Processing actions

```
do
{
    DSTREG      = Memory[DR] – Memory[SR];
    if (DSTREG != 0)
    {
        break;
    }
    DR          = DR + 1; // DR: Destination Register (R13)
    SR          = SR + 1; // SR: Source Register (R12)
    CR          = CR - 1; // CR: Count Register (R11)
    IP          = IP - 1; // IP: InstructionPointer internal register
} while (CR > 0);
```

## CIF

Convert Integer to Float

### Description

CIF interprets the specified register  as an integer value. Then converts
that value  to a float representation  and stores the result  back in the
same register.

Note that, due to the limited precision of the float representation, high
enough values  of a 32-bit integer  will result in a  precision loss when
represented as a float.

### Variants and Actions

| Form          | Processing Action                            |
| ------------- | -------------------------------------------- |
| ```CIF REG``` | ```REG = (float) REG;```                     |

## CFI

Convert Float to Integer

### Description

CFI interprets  the specified  register as a  float value.  Then converts
that value to an integer representation and stores the result back in the
same register.  Conversion is not  done through rounding, but  instead by
truncating (the fractional part is discarded).

Note that,  due to the  much greater  range of the  float representation,
high  enough values  of a  float  will result  in a  precision loss  when
represented as a 32-bit integer.

### Variants and Actions

| Form          | Processing Action                            |
| ------------- | -------------------------------------------- |
| ```CFI REG``` | ```REG = (int) REG;```                       |

## CIB

Convert Integer to Boolean

### Description

CIB interprets the specified register  as an integer value. Then converts
that value to  its standard boolean representation and  stores the result
back in  the same register. This  means that all non-zero  values will be
converted to 1.

### Variants and Actions

| Form          | Processing Action               |
| ------------- | ------------------------------- |
| ```CIB REG``` | ```REG = (REG != 0) ? 1 : 0;``` |

## CFB

Convert Float to Boolean

### Description

CFB interprets  the specified  register as a  float value.  Then converts
that value to either 0 (for float  value 0.0), or 1 (for any other value)
and stores it back in that register.

### Variants and Actions

| Form          | Processing Action                 |
| ------------- | --------------------------------- |
| ```CFB REG``` | ```REG = (REG != 0.0) ? 1 : 0;``` |

## NOT

Bitwise NOT

### Description

NOT  performs a  binary ‘not’  by inverting  all of  the bits  in the
specified register.

### Variants and Actions

| Form          | Processing Action |
| ------------- | ----------------- |
| ```NOT REG``` | ```REG = ~REG;``` |

## AND

Bitwise AND

### Description

AND performs  a **Bitwise AND** between  each pair of respective  bits in
the 2  specified operands.  The result  is stored in  the first  of them,
which is always a register.

### AND truth table

| A     | B     | X     |
| ----- | ----- | ----- |
| false | false | false |
| false | true  | false |
| true  | false | false |
| true  | true  | true  |

### Variants and Actions

| Form                        | Processing Action                  |
| --------------------------- | ---------------------------------- |
| ```AND DSTREG, Immediate``` | ```DSTREG = DSTREG & Immediate;``` |
| ```AND DSTREG, SRCREG```    | ```DSTREG = DSTREG & SRCREG;```    |

## OR

Bitwise inclusive OR (iOR)

### Description

OR performs  a **Bitwise INCLUSIVE  OR** between each pair  of respective
bits in the  2 specified operands. The  result is stored in  the first of
them, which is always a register.

### iOR truth table

| A     | B     | X     |
| ----- | ----- | ----- |
| false | false | false |
| false | true  | true  |
| true  | false | true  |
| true  | true  | true  |

### Variants and Actions

| Form                       | Processing Action                   |
| -------------------------- | ----------------------------------- |
| ```OR DSTREG, Immediate``` | ```DSTREG = DSTREG \| Immediate;``` |
| ```OR DSTREG, SRCREG```    | ```DSTREG = DSTREG \| SRCREG;```    |

## XOR

Bitwise exclusive OR (XOR)

### Description

XOR performs a  **Bitwise EXCLUSIVE OR** between each  pair of respective
bits in the  2 specified operands. The  result is stored in  the first of
them, which is always a register.

### XOR truth table

| A     | B     | X     |
| ----- | ----- | ----- |
| false | false | false |
| false | true  | true  |
| true  | false | true  |
| true  | true  | false |

### Variants and Actions

| Form                        | Processing Action                  |
| --------------------------- | ---------------------------------- |
| ```XOR DSTREG, Immediate``` | ```DSTREG = DSTREG ^ Immediate;``` |
| ```XOR DSTREG, SRCREG```    | ```DSTREG = DSTREG ^ SRCREG;```    |

## BNOT

boolean NOT

### Description

BNOT interprets the specified register as  a boolean and then converts it
to the opposite boolean value. This  is equivalent to first using CIB and
then inverting bit number 0.

### Variants and Actions

| Form           | Processing Action               |
| -------------- | ------------------------------- |
| ```BNOT REG``` | ```REG = (REG == 0) ? 1 : 0;``` |

### Errata

Prior to release **v25.10.29**, there was  a bug in the compiler where in
some contexts,  a bitwise NOT would  be erroneously applied instead  of a
boolean NOT. Upgrading to v25.10.29 or later fixes this issue.

## SHL

Bitwise shift left

### Description

SHL performs  an bit  shift to  the left in  the specified  register. The
second operand is taken as an integer number of positions to shift.

Shifting  0 positions  has no  effect,  while negative  values result  in
shifting right.

The shift  type is  logical: in  shifts left,  overflow is  discarded and
zeroes are introduced as least significant bits.

In shifts right, underflow is discarded and zeroes are introduced as most
significant bits.

### Variants and Actions

| Form                        | Processing Action                   |
| --------------------------- | ----------------------------------- |
| ```SHL DSTREG, Immediate``` | ```DSTREG = DSTREG << Immediate;``` |
| ```SHL DSTREG, SRCREG```    | ```DSTREG = DSTREG << SRCREG;```    |

## IADD

Integer Addition

### Description

IADD  interprets  both  of  its  operands as  integers  and  performs  an
addition. The  result is stored in  the first operand, which  is always a
register. Overflow bits are discarded.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```IADD DSTREG, Immediate``` | ```DSTREG = DSTREG + Immediate``` |
| ```IADD DSTREG, SRCREG```    | ```DSTREG = DSTREG + SRCREG```    |

## ISUB

Integer Subtraction

### Description

ISUB  interprets  both  of  its  operands  as  integers  and  performs  a
subtraction. The result is stored in the first operand, which is always a
register. Overflow bits are discarded.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```ISUB DSTREG, Immediate``` | ```DSTREG = DSTREG - Immediate``` |
| ```ISUB DSTREG, SRCREG```    | ```DSTREG = DSTREG - SRCREG```    |

## IMUL

Integer Multiplication

### Description

IMUL  interprets  both  of  its  operands  as  integers  and  performs  a
multiplication.  The result  is stored  in  the first  operand, which  is
always a register. Overflow bits are discarded.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```IMUL DSTREG, Immediate``` | ```DSTREG = DSTREG * Immediate``` |
| ```IMUL DSTREG, SRCREG```    | ```DSTREG = DSTREG * SRCREG```    |

## IDIV

Integer Division

### Description

IDIV interprets both of its operands as integers and performs a division.
The result (quotient)  is stored in the first operand,  which is always a
register.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```IDIV DSTREG, Immediate``` | ```DSTREG = DSTREG / Immediate``` |
| ```IDIV DSTREG, SRCREG```    | ```DSTREG = DSTREG / SRCREG```    |

## IMOD

Integer Modulus

### Description

IMOD interprets both of its operands as integers and performs a division.
The remainder of  that division is stored in the  first operand, which is
always a register.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```IMOD DSTREG, Immediate``` | ```DSTREG = DSTREG % Immediate``` |
| ```IMOD DSTREG, SRCREG```    | ```DSTREG = DSTREG % SRCREG```    |

## ISGN

Integer Sign Change

### Description

ISGN interprets the operand register as an integer and inverts its sign.

### Variants and Actions

| Form           | Processing Action |
| -------------- | ----------------- |
| ```ISGN REG``` | ```REG = -REG```  |

## IMIN

Integer Minimum

### Description

IMIN  interprets both  of its  operands as  integers. It  then takes  the
minimum (smaller)  of both  values and  stores it  in the  first operand,
which is always a register.

### Variants and Actions

| Form                         | Processing Action                    |
| ---------------------------- | ------------------------------------ |
| ```IMIN DSTREG, Immediate``` | ```DSTREG = min(DSTREG, Immediate``` |
| ```IMIN DSTREG, SRCREG```    | ```DSTREG = min(DSTREG, SRCREG)``    |

## IMAX

Integer Maximum

### Description

IMAX  interprets both  of its  operands as  integers. It  then takes  the
maximum (larger) of both values and stores it in the first operand, which
is always a register.

### Variants and Actions

| Form                         | Processing Action                    |
| ---------------------------- | ------------------------------------ |
| ```IMAX DSTREG, Immediate``` | ```DSTREG = max(DSTREG, Immediate``` |
| ```IMAX DSTREG, SRCREG```    | ```DSTREG = max(DSTREG, SRCREG)``    |

## IABS

Integer Absolute Value

### Description

IABS interprets the operand register as an integer and takes its absolute value.

### Variants and Actions

| Form           | Processing Action     |
| -------------- | --------------------- |
| ```IABS REG``` | ```REG = abs(REG)```  |

## FADD

Float Addition

### Description

FADD interprets both of its operands  as floats and performs an addition.
The result  is stored in the  first operand, which is  always a register.
Overflow bits are discarded.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```FADD DSTREG, Immediate``` | ```DSTREG = DSTREG + Immediate``` |
| ```FADD DSTREG, SRCREG```    | ```DSTREG = DSTREG + SRCREG```    |

## FSUB

Float Subtraction

### Description

FSUB  interprets  both   of  its  operands  as  floats   and  performs  a
subtraction. The result is stored in the first operand, which is always a
register. Overflow bits are discarded.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```FSUB DSTREG, Immediate``` | ```DSTREG = DSTREG - Immediate``` |
| ```FSUB DSTREG, SRCREG```    | ```DSTREG = DSTREG - SRCREG```    |

## FMUL

Float Multiplication

### Description

FMUL  interprets  both   of  its  operands  as  floats   and  performs  a
multiplication.  The result  is stored  in  the first  operand, which  is
always a register. Overflow bits are discarded.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```FMUL DSTREG, Immediate``` | ```DSTREG = DSTREG * Immediate``` |
| ```FMUL DSTREG, SRCREG```    | ```DSTREG = DSTREG * SRCREG```    |

## FDIV

Float Division

### Description

FDIV interprets both  of its operands as floats and  performs a division.
The result (quotient)  is stored in the first operand,  which is always a
register.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```FDIV DSTREG, Immediate``` | ```DSTREG = DSTREG / Immediate``` |
| ```FDIV DSTREG, SRCREG```    | ```DSTREG = DSTREG / SRCREG```    |

## FMOD

Float Modulus (Remainder from Division)

### Description

FMOD interprets  both of its operands  as floats and performs  a floating
point division.  It then takes  the remainder  of that division  when the
result’s  fractional part  is  discarded  and stores  it  in the  first
operand, which is always a register.

### Variants and Actions

| Form                         | Processing Action                 |
| ---------------------------- | --------------------------------- |
| ```FMOD DSTREG, Immediate``` | ```DSTREG = DSTREG % Immediate``` |
| ```FMOD DSTREG, SRCREG```    | ```DSTREG = DSTREG % SRCREG```    |

## FSGN

Float Sign Change

### Description

FSGN interprets the operand register as a float and inverts its sign.

### Variants and Actions

| Form           | Processing Action |
| -------------- | ----------------- |
| ```FSGN REG``` | ```REG = -REG```  |

## FMIN

Float Minimum

### Description

FMIN interprets both of its operands as floats. It then takes the minimum
(smaller) of  both values and  stores it in  the first operand,  which is
always a register.

### Variants and Actions

| Form                         | Processing Action                    |
| ---------------------------- | ------------------------------------ |
| ```FMIN DSTREG, Immediate``` | ```DSTREG = min(DSTREG, Immediate``` |
| ```FMIN DSTREG, SRCREG```    | ```DSTREG = min(DSTREG, SRCREG)``    |

## FMAX

Float Maximum

### Description

FMAX interprets both of its operands as floats. It then takes the maximum
(larger) of  both values  and stores  it in the  first operand,  which is
always a register.

### Variants and Actions

| Form                         | Processing Action                    |
| ---------------------------- | ------------------------------------ |
| ```FMAX DSTREG, Immediate``` | ```DSTREG = max(DSTREG, Immediate``` |
| ```FMAX DSTREG, SRCREG```    | ```DSTREG = max(DSTREG, SRCREG)``    |

## FABS

Floating Point Absolute Value

### Description

FABS interprets  the operand register as  a float and takes  its absolute
value.

### Variants and Actions

| Form           | Processing Action     |
| -------------- | --------------------- |
| ```FABS REG``` | ```REG = abs(REG)```  |

## FLR

Floating Point Round Down (Floor)

### Description

FLR interprets the operand register as a float and rounds it downwards to
an integer  value. Note that the  result is not converted  to an integer,
but is still a float.

### Variants and Actions

| Form          | Processing Action       |
| ------------- | ----------------------- |
| ```FLR REG``` | ```REG = floor(REG)```  |

## CEIL

Floating Point Round Up (Ceiling)

### Description

CEIL interprets the operand register as  a float and rounds it upwards to
an integer  value. Note that the  result is not converted  to an integer,
but is still a float.

### Variants and Actions

| Form           | Processing Action      |
| -------------- | ---------------------- |
| ```CEIL REG``` | ```REG = ceil(REG)```  |

## ROUND

Floating Point Round to Nearest Whole Value

### Description

ROUND interprets  the operand register  as a float  and rounds it  to the
closest  integer value.  Note  that the  result is  not  converted to  an
integer, but is still a float.

### Variants and Actions

| Form            | Processing Action       |
| --------------- | ----------------------- |
| ```ROUND REG``` | ```REG = round(REG)```  |

## SIN

Calculate the Sine

### Description

SIN interprets the operand register as a float and calculates the sine of
that value. The sine function will interpret its argument in radians.

### Variants and Actions

| Form          | Processing Action     |
| ------------- | --------------------- |
| ```SIN REG``` | ```REG = sin(REG)```  |

## ACOS
Arc cosine

### Description

ACOS interprets  the operand register as  a float and calculates  the arc
cosine of that value.  The result is given in radians, in  the range 0 to
PI.

### Variants and Actions

| Form           | Processing Action      |
| -------------- | ---------------------- |
| ```ACOS REG``` | ```REG = acos(REG);``` |

## ATAN2

Arc Tangent from x and y

### Description

ATAN2  interprets both  operand registers  as floats  and calculates  the
angle of a  vector such that Vx =  SRCREG and Vy = DSTREG.  The result is
stored in the first operand register and will be given in radians, in the
range of -PI to PI.  The origin of angles is located at (Vx  > 0, Vy = 0)
and angles grow when rotating towards (Vx = 0, Vy > 0).

### Variants and Actions

| Form                       | Processing Action                  |
| -------------------------- | ---------------------------------- |
| ```ATAN2 DSTREG, SRCREG``` | ```REG = atan2(DSTREG, SRCREG);``` |

## LOG

Natural logarithm

### Description

LOG  interprets  the operand  register  as  a  float and  calculates  the
logarithm base e of that value.

### Variants and Actions

| Form          | Processing Action     |
| ------------- | --------------------- |
| ```LOG REG``` | ```REG = log(REG);``` |

## POW

Raise to a Power

### Description

POW interprets both operand registers as floats and calculates the result
of raising  the first  operand to  the power of  the second  operand. The
result is stored in the first operand register.

### Variants and Actions

| Form                     | Processing Action                |
| ------------------------ | -------------------------------- |
| ```POW DSTREG, SRCREG``` | ```REG = pow(DSTREG, SRCREG);``` |

## Vircon32 ROM Specification

The binary formats of the Vircon32 ROM/Cartridge are as follows:

### packed V32 ROM

!(V32 ROM header)[V32_ROM_header.png]

Once assembled into object form, and packed together with any graphics or
audio assets, we have this final ROM. The general layout is as follows:

!(V32 ROM layout)[V32_ROM_layout.png]

### assembled VBIN

Once code  is assembled,  we have  the Vircon32  equivalent of  an object
file, known as a **VBIN** file. Its header is as follows:

!(V32 VBIN header)[V32_VBIN_header.png]

Note the word offsets (and how a word is 4 bytes). This will be useful to
calculate any offsets when debugging.

### texture VTEX

Any image assets, once processed, are stored in the Vircon32 VTEX format,
which has a header as follows:

!(V32 VTEX header)[V32_VTEX_header.png]
