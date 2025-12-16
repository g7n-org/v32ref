# v32ref: Vircon32 assembly language reference

An information-dense, quick-reference guide to Vircon32 system details and assembly language mneumonics to aid in development.

This document is based on Vircon32 DevTools **v25.1.19** or later; older versions will contain inconsistencies.

## table of contents

  * [System Quick Reference](#system-quick-reference)
  * [Assembler Data Directives](#assember-data-directives)
  * [Vircon32 Instruction Set Category Overview](#vircon32-instruction-set-category-overview)
  * [Vircon32 Instruction Set Detailed View](#vircon32-instruction-set-detailed-view)
  * [Vircon32 Memory Map](#vircon32-memory-map)
  * [Vircon32 IOPorts Layout](#vircon32-ioports-layout)
  * [IOPorts](#ioports)
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
  * [Instruction Format](#instruction-format)

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
There  are 64  CPU opcodes,  instructions  encode them  in 6  bits. No invalid opcodes  can exist. HLT  is opcode 0 for  safety: if an  empty or invalid instruction is found, the CPU will stop execution.

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

# IOPorts

## TIME

| Type | Port  | Name             | Description                  |
| ---- | ----- | ---------------- | ---------------------------- |
| IN   | 0x000 | TIM_CurrentDate  | retrieve current date        |
| IN   | 0x001 | TIM_CurrentTime  | retrieve current time        |
| IN   | 0x002 | TIM_FrameCounter | retrieve current frame count |
| IN   | 0x003 | TIM_CycleCounter | retrieve current cycle count |

### example: get current frame count

```
    in R0,  TIM_FrameCounter    ; load current frame count into R0
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

## Instruction Format

{{:notes:comporg:spring2025:v32xx_instruction_format.jpeg|}}

  * bits 31-26: opcode
  * bit 25: immediate value
  * bits 24-21: register 1
  * bits 20-17: register 2
  * bits 16-14: address mode
  * bits 13-0: port number

If the **immediate value** bit is set, an additional word is read to be used as a parameter to the instruction.

## HLT

## WAIT

## JMP
Unconditional jump. Forcibly redirect program flow to indicated address. The address is somewhere else in the program logic, likely identified by some set label.

### Structure and variants

  * Variant 1: <code>JMP { ImmediateValue }</code>
  * Variant 2: <code>JMP { Register1 }</code>

### Processing actions
  * Variant 1: <code>InstructionPointer = ImmediateValue</code>
  * Variant 2: <code>InstructionPointer = Register1</code>

### Description
**JMP** performs an unconditional jump to the address specified by its operand. After processing this instruction the CPU will continue execution at the new address.

### Examples
Jumping to a label (memory address/offset):

<code>
    jmp _label
    ...
_label:
</code>

Jumping to address stored in register:

<code>
    jmp R0
</code>

====CALL====

====RET====

====JT====
Jump if True: a conditional jump typically used following a comparison instruction, should the queried register contain a true (1) value, jump to indicated address.

===NOTE===
For the purposes of comparisons and conditional jumps on Vircon32:

  * true is 1 (technically non-zero)
  * false is 0

===Structure and variants===
  * Variant 1: <code>JT { Register1 }, { ImmediateValue }</code>
  * Variant 2: <code>JT { Register1 }, { Register2 }</code>

===Effect===

  * Variant 1: <code>if Register1 != 0 then InstructionPointer = ImmediateValue</code>
  * Variant 2: <code>if Register1 != 0 then InstructionPointer = Register2</code>

===Description===
JT performs a jump only if its first operand is true, i.e. non zero when taken as an integer. In that case its behavior is the same as an unconditional jump. Otherwise it has no effect.

====JF====
Jump if False: a conditional jump typically used following a comparison instruction, should the queried register contain a false (0) value, jump to indicated address.

===NOTE===
For the purposes of comparisons and conditional jumps on Vircon32:

  * true is 1 (technically non-zero)
  * false is 0

===Structure and variants===
  * Variant 1: <code>JF { Register1 }, { ImmediateValue }</code>
  * Variant 2: <code>JF { Register1 }, { Register2 }</code>

===Effect===

  * Variant 1: <code>if Register1 == 0 then InstructionPointer = ImmediateValue</code>
  * Variant 2: <code>if Register1 == 0 then InstructionPointer = Register2</code>

===Description===
JF performs a jump only if its first operand is false, i.e. zero when taken as an integer. In that case its behavior is the same as an unconditional jump. Otherwise it has no effect.

====Integer Comparisons====
For the purposes of comparisons and conditional jumps on Vircon32:

  * true is 1 (technically non-zero)
  * false is 0

There are six relational operations:

  * is equal to
  * is not equal to
  * is less than
  * is than or equal to
  * is greater than
  * is greater than or equal to

====IEQ====
Integer Compare Equality: comparisons allow us typically to evaluate two values, in accordance with some relational operation, resulting in a true (1) or false (0) result.

Should the first operand contain the same information as the second operand, the result will be true. Otherwise, false.

===Structure and variants===
  * Variant 1: <code>IEQ { Register1 }, { ImmediateValue }</code>
  * Variant 2: <code>IEQ { Register1 }, { Register2 }</code>

===Processing actions===
  * Variant 1: <code>if Register1 == ImmediateValue then Register1 = 1 else Register1 = 0</code>
  * Variant 2: <code>if Register1 == Register2 then Register1 = 1 else Register1 = 0</code>

===Description===
IEQ takes two operands interpreted as integers, and checks if they are equal. It will store the boolean result in the first operand, which is always a register.

====INE====
Integer Not Equal: comparisons allow us typically to evaluate two values, in accordance with some relational operation, resulting in a true (1) or false (0) result.

Here, we test to see if the first operand is not equal to the second operand. If they are equal, the result is false, otherwise, not being equal yields a result of true.

===Structure and variants===
  * Variant 1: <code>INE { Register1 }, { ImmediateValue }</code>
  * Variant 2: <code>INE { Register1 }, { Register2 }</code>

===Processing actions===
  * Variant 1: <code>if Register1 != ImmediateValue then Register1 = 1 else Register1 = 0</code>
  * Variant 2: <code>if Register1 != Register2 then Register1 = 1 else Register1 = 0</code>

===Description===
INE takes two operands interpreted as integers, and checks if they are different. It will store the boolean result in the first operand, which is always a register.

====IGT====
Integer Greater Than: comparisons allow us typically to evaluate two values, in accordance with some relational operation, resulting in a true (1) or false (0) result.

In this case, we are testing if the first operand is greater than the second operand.

===Structure and variants===
  * Variant 1: <code>IGT { Register1 }, { ImmediateValue }</code>
  * Variant 2: <code>IGT { Register1 }, { Register2 }</code>

===Processing actions===
  * Variant 1: <code>if Register1 > ImmediateValue then Register1 = 1 else Register1 = 0</code>
  * Variant 2: <code>if Register1 > Register2 then Register1 = 1 else Register1 = 0</code>

===Description===
IGT takes two operands interpreted as integers, and checks if the first one is greater than the second. It will store the boolean result in the first operand, which is always a register.


====IGE====
Integer Greater Than Or Equal: comparisons allow us typically to evaluate two values, in accordance with some relational operation, resulting in a true (1) or false (0) result.

In this case, we are testing if the first operand is greater than or equal to the second operand.

===Structure and variants===
  * Variant 1: <code>IGE { Register1 }, { ImmediateValue }</code>
  * Variant 2: <code>IGE { Register1 }, { Register2 }</code>

===Processing actions===
  * Variant 1: <code>if Register1 >= ImmediateValue then Register1 = 1 else Register1 = 0</code>
  * Variant 2: <code>if Register1 >= Register2 then Register1 = 1 else Register1 = 0</code>

===Description===
IGE takes two operands interpreted as integers, and checks if the first one is greater or equal to the second. It will store the boolean result in the first operand, which is always a register.

====ILT====
Integer Less Than: comparisons allow us typically to evaluate two values, in accordance with some relational operation, resulting in a true (1) or false (0) result.

In this case, we are testing if the first operand is less than the second operand.

===Structure and variants===
  * Variant 1: <code>ILT { Register1 }, { ImmediateValue }</code>
  * Variant 2: <code>ILT { Register1 }, { Register2 }</code>

===Processing actions===
  * Variant 1: <code>if Register1 < ImmediateValue then Register1 = 1 else Register1 = 0</code>
  * Variant 2: <code>if Register1 < Register2 then Register1 = 1 else Register1 = 0</code>

===Description===
ILT takes two operands interpreted as integers, and checks if the first one is less than the second. It will store the boolean result in the first operand, which is always a register.

====ILE====
Integer Less Than Or Equal: comparisons allow us typically to evaluate two values, in accordance with some relational operation, resulting in a true (1) or false (0) result.

In this case, we are testing if the first operand is less than or equal to the second operand.

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>ILE DSTREG, ImmediateValue</code>  |<code c>if (DSTREG <= ImmediateValue)
    DSTREG=1;
else
    DSTREG=0;</code>  |
|  2  |<code asm>ILE DSTREG, SRCREG</code>  |<code c>if (DSTREG <= SRCREG)
    DSTREG=1;
else
    DSTREG=0;</code>  |

===Description===
ILE takes two operands interpreted as integers, and checks if the first one is less or equal to the second. It will store the boolean result in the first operand, which is always a register.

====FEQ====
====FNE====
====FGT====
====FGE====
====FLT====
====FLE====

====MOV====
MOVE: your general purpose data-copying instruction.

===Addressing===
MOVE, like other data-centric instructions, makes use of various addressing modes:

  * **register**: source/destination is an inbuilt CPU register
  * **immediate**: some literal constant (be it data or a memory address)
  * **indirect**: value isn't the data, but a memory address to where the data is. Think pointer dereference. It comes in 3 varieties:
  * **indexed**: an offset to some existing piece of data.
      * **immediate**: a literal constant (data or memory address)
      * **indexed**: used with immediate/register, but we can do additional math to get an offset from the address. Think pointer dereference on an array.
      * **register**: a CPU register

Indirect processing is accomplished with the **<nowiki>[ ]</nowiki>** (square brackets) surrounding the value we wish to dereference (we're not interested in the direct thing, but indirectly in what that thing contains).

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>MOV DSTREG, ImmediateValue</code>  |<code c>DSTREG = ImmediateValue;</code>  |
|  2  |<code asm>MOV DSTREG, SRCREG</code>  |<code c>DSTREG = SRCREG;</code>  |
|  3  |<code asm>MOV DSTREG, [ImmediateValue]</code>  |<code c>DSTREG = Memory[ImmediateValue];</code>  |
|  4  |<code asm>MOV DSTREG, [SRCREG]</code>  |<code c>DSTREG = Memory[SRCREG];</code>  |
|  5  |<code asm>MOV DSTREG, [SRCREG+ImmediateValue]</code>  |<code c>DSTREG = Memory[SRCREG+ImmediateValue];</code>  |
|  6  |<code asm>MOV [ImmediateValue], SRCREG</code>  |<code c>Memory[ImmediateValue] = SRCREG;</code>  |
|  7  |<code asm>MOV [DSTREG], SRCREG</code>  |<code c>Memory[DSTREG] = SRCREG;</code>  |
|  8  |<code asm>MOV [DSTREG+ImmediateValue], SRCREG</code>  |<code c>Memory[DSTREG+ImmediateValue] = SRCREG;</code>  |

===Description===
MOV copies the value indicated in its second operand into the register or memory address indicated by its first operand. MOV is the most complex instruction to process because it needs to distinguish between 8 different addressing modes.

The instruction specifies which of the 8 modes to use in its “Addressing mode” field, being the possible values interpreted as follows:

==MOV Addressing modes==

^  Binary  ^  Destination  ^  Source  |
|  000  |  DSTREG  |  Immediate Value  |
|  001  |  DSTREG  |  SRCREG  |
|  010  |  DSTREG  |  Memory <nowiki>[Immediate Value]</nowiki>  |
|  011  |  DSTREG  |  Memory <nowiki>[SRCREG]</nowiki>  |
|  100  |  DSTREG  |  Memory <nowiki>[SRCREG + Immediate Value]</nowiki>  |
|  101  |  <nowiki>Memory[Immediate Value]</nowiki>  |  SRCREG  |
|  110  |  <nowiki>Memory[DSTREG]</nowiki>  |  SRCREG  |
|  111  |  <nowiki>Memory[DSTREG + Immediate Value]</nowiki>  |  SRCREG  |

====LEA====
Load Effective Address of a memory position.

===Structure and variants===
  * Variant 1: <code>LEA { Register1 }, [ { Register2 } ]</code>
  * Variant 2: <code>LEA { Register1 }, [ { Register2 } + { ImmediateValue } ]</code>

===Processing actions===
  * Register: <code>Register1 = Register2</code>
  * Indexed: <code>Register1 = Register2 + ImmediateValue</code>

===Description===
LEA takes a memory address as second operand. It stores that address (not its contents) into the register given as first operand. The most useful case is when the address is given in the form pointer + offset, since the addition is automatically performed.

====PUSH====
Save to top of stack

===Structure and variants===
  * <code>PUSH { Register1 }</code>

===Processing actions===
  * <code>Stack.Push(Register1)</code>

===Description===
PUSH uses the CPU hardware stack to add the value contained in the given register at the top of the stack. When you PUSH a value onto the stack, the STACK POINTER (SP) is adjusted downward by one address offset (stack grows down).

====POP====
Load from top of stack

===Structure and variants===
  * <code>POP { Register1 }</code>

===Processing actions===
  * <code>Register1 = Stack.Pop()</code>

===Description===
POP uses the CPU hardware stack to remove a value from the top of the stack and write it in the given register. When you POP a value off the stack, the STACK POINTER (SP) is adjusted upward by one address offset (stack grows down, shrinks up).

====IN====
Receive input from an I/O port

===Structure and variants===
  * <code>IN {Register1}, {PortNumber}</code>

===Processing actions===
  * <code>Register1 = Port[ PortNumber ]</code>

===Description===
In uses the control bus to read from an I/O port in another chip and stores the returned value in the specified register. This read request may lead to side effects depending on the specified port.

====OUT====
Write to an I/O port

===Structure and variants===
  * <code>OUT {PortNumber}, [{ImmediateValue}]</code>
  * <code>OUT {PortNumber}, {Register1}</code>

===Processing actions===
  * <code>Port[PortNumber] = ImmediateValue</code>
  * <code>Port[PortNumber] = Register1</code>

===Description===
OUT uses the control bus to write the specified value to an I/O port in another chip. This write request may lead to side effects depending on the specified port

====MOVS====
Copy string (HW memcpy)

===Structure and variants===
  * <code>MOVS</code>

===Processing actions===
  * <code>Memory[ DR ] = Memory[ SR ]</code>
  * <code>DR += 1</code>
  * <code>SR += 1</code>
  * <code>CR -= 1</code>
  * <code>if CR > 0 then InstructionPointer -= 1</code>

===Description===
MOVS copies a value from the memory address pointed by SR to the one pointed by DR
(as in a supposed MOV [DR], [SR]). It then implements a local loop to repeat itself until
the counter in CR reaches 0, while working on consecutive addresses.
Note that even when called with a value of CR of zero or less, MOVS will always perform
the described loop at least once.
This instruction is the only way in Vircon32 CPU to directly copy values from 2 places in
memory without going through a register.

====SETS====
Set string (HW memset)

===Structure and variants===
  * <code>SETS</code>

===Processing actions===
  * <code>Memory[ DR ] = SR</code>
  * <code>DR += 1</code>
  * <code>CR -= 1</code>
  * <code>if CR > 0 then InstructionPointer -= 1</code>

===Description===
SETS copies the value in SR to the address pointed by DR (as in a MOV [DR], SR). It
then implements a local loop to repeat itself until the counter in CR reaches 0, while
writing to consecutive addresses.
Note that even when called with a value of CR of zero or less, SETS will always perform
the described loop at least once.

====CMPS====
Compare string (HW memcmp)

===Structure and variants===
  * <code>CMPS { Register1 }</code>

===Processing actions===
  * <code>Register1 = Memory[ DR ] – Memory[ SR ]</code>
  * <code>if Register1 != 0 then end processing</code>
  * <code>DR += 1</code>
  * <code>SR += 1</code>
  * <code>CR -= 1</code>
  * <code>if CR > 0 then InstructionPointer -= 1</code>

===Description===
CMPS takes as a reference the compares the value in the address pointed by DR and
compares it with the one pointed by SR, by subtracting. It then implements a local loop
to repeat itself until the counter in CR reaches 0, while reading consecutive addresses.
The comparison result will be stored in the specified register, and will be zero when
equal, positive when some value at [DR] was greater, and negative when some value in
[SR] was greater.
Note that even when called with a value of CR of zero or less, CMPS will always perform
the described loop at least once.

====CIF====
Convert Integer to Float

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>CIF DSTREG</code>  |<code c>DSTREG = (float)DSTREG;</code>  |

===Description===
CIF interprets the specified register as an integer value. Then converts that value to a
float representation and stores the result back in the same register.
Note that, due to the limited precision of the float representation, high enough values of
a 32-bit integer will result in a precision loss when represented as a float.
====CFI====
Convert Float to Integer

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>CFI DSTREG</code>  |<code c>DSTREG = (int)DSTREG;</code>  |

===Description===
CFI interprets the specified register as a float value. Then converts that value to an
integer representation and stores the result back in the same register. Conversion is not
done through rounding, but instead by truncating (the fractional part is discarded).
Note that, due to the much greater range of the float representation, high enough values
of a float will result in a precision loss when represented as a 32-bit integer.

====CIB====
Convert Integer to Boolean

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>CIB DSTREG</code>  |<code c>if (DSTREG != 0)
    DSTREG = 1;
else
    DSTREG = 0;</code>  |


===Description===
CIB interprets the specified register as an integer value. Then converts that value to its
standard boolean representation and stores the result back in the same register. This
means that all non-zero values will be converted to 1.

====CFB====
Convert Float to Boolean

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>CFB DSTREG</code>  |<code c>if (DSTREG != 0.0)
    DSTREG = 1;
else
    DSTREG = 0;</code>  |

===Description===
CFB interprets the specified register as a float value. Then converts that value to either 0
(for float value 0.0), or 1 (for any other value) and stores it back in that register.

====NOT====
Bitwise NOT

===Structure and variants===
  * <code>NOT { Register1 }</code>

===Processing actions===
  * <code>Register1 = NOT Register1</code>

===Description===
NOT performs a binary ‘not’ by inverting all of the bits in the specified register.

====AND====
Bitwise AND

===AND truth table===

^  A  ^  B  ^  X  |
|  false  |  false  |  false  |
|  false  |  true  |  false  |
|  true  |  false  |  false  |
|  true  |  true  |  true  |

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>AND DSTREG, ImmediateValue</code>  |<code c>DSTREG = DSTREG & ImmediateValue;</code>  |
|  2  |<code asm>AND DSTREG, SRCREG</code>  |<code c>DSTREG = DSTREG & SRCREG;</code>  |

===Description===
AND performs a **Bitwise AND** between each pair of respective bits in the 2 specified
operands. The result is stored in the first of them, which is always a register.

====OR====
Bitwise iOR

===iOR truth table===

^  A  ^  B  ^  X  |
|  false  |  false  |  false  |
|  false  |  true  |  true  |
|  true  |  false  |  true  |
|  true  |  true  |  true  |

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>OR DSTREG, ImmediateValue</code>  |<code c>DSTREG = DSTREG | ImmediateValue;</code>  |
|  2  |<code asm>OR DSTREG, SRCREG</code>  |<code c>DSTREG = DSTREG | SRCREG;</code>  |

===Description===
OR performs a **Bitwise INCLUSIVE OR** between each pair of respective bits in the 2 specified operands. The result is stored in the first of them, which is always a register.

====XOR====
Bitwise XOR

===XOR truth table===

^  A  ^  B  ^  X  |
|  false  |  false  |  false  |
|  false  |  true  |  true  |
|  true  |  false  |  true  |
|  true  |  true  |  false  |

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>XOR DSTREG, ImmediateValue</code>  |<code c>DSTREG = DSTREG ^ ImmediateValue;</code>  |
|  2  |<code asm>XOR DSTREG, SRCREG</code>  |<code c>DSTREG = DSTREG ^ SRCREG;</code>  |

===Description===
XOR performs a **Bitwise EXCLUSIVE OR** between each pair of respective bits in the 2
specified operands. The result is stored in the first of them, which is always a register.

====BNOT====
Boolean NOT

===Structure and variants===
  * <code>BNOT { Register1 }</code>

===Processing actions===
  * <code>if Register1 == 0 then Register1 = 1 else Register1 = 0</code>

===Description===
BNOT interprets the specified register as a boolean and then converts it to the opposite
boolean value. This is equivalent to first using CIB and then inverting bit number 0.

====SHL====
Bit shift left

===Structure and variants===
^  Variant  ^  Form  ^  Action  |
|  1  |<code asm>SHL DSTREG, ImmediateValue</code>  |<code c>DSTREG = DSTREG << ImmediateValue;</code>  |
|  2  |<code asm>SHL DSTREG, SRCREG</code>  |<code c>DSTREG = DSTREG << SRCREG;</code>  |

===Description===
SHL performs an bit shift to the left in the specified register. The second operand is
taken as an integer number of positions to shift.

Shifting 0 positions has no effect, while negative values result in shifting right.

The shift type is logical: in shifts left, overflow is discarded and zeroes are introduced as least significant bits.

In shifts right, underflow is discarded and zeroes are introduced as most significant bits.


====IADD====
Integer Addition

===Structure and variants===
  * <code>(Variant 1): IADD { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): IADD { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 += ImmediateValue</code>
  * <code>(Variant 2): Register1 += Register2</code>

===Description===
IADD interprets both of its operands as integers and performs an addition. The result is
stored in the first operand, which is always a register. Overflow bits are discarded.

====ISUB====
Integer Subtraction

===Structure and variants===
  * <code>(Variant 1): ISUB { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): ISUB { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 -= ImmediateValue</code>
  * <code>(Variant 2): Register1 -= Register2</code>

===Description===
ISUB interprets both of its operands as integers and performs a subtraction. The result
is stored in the first operand, which is always a register. Overflow bits are discarded.

====IMUL====
Integer Multiplication

===Structure and variants===
  * <code>(Variant 1): IMUL { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): IMUL { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 *= ImmediateValue</code>
  * <code>(Variant 2): Register1 *= Register2</code>

===Description===
IMUL interprets both of its operands as integers and performs a multiplication. The
result is stored in the first operand, which is always a register. Overflow bits are
discarded.

====IDIV====
Integer Division

===Structure and variants===
  * <code>(Variant 1): IDIV { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): IDIV { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 /= ImmediateValue</code>
  * <code>(Variant 2): Register1 /= Register2</code>

===Description===
IDIV interprets both of its operands as integers and performs a division. The result is
stored in the first operand, which is always a register.

====IMOD====
Integer Modulus

===Structure and variants===
  * <code>(Variant 1): IMOD { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): IMOD { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 = Register1 mod ImmediateValue</code>
  * <code>(Variant 2): Register1 = Register1 mod Register2</code>

===Description===
IMOD interprets both of its operands as integers and performs a division. The remainder
of that division is stored in the first operand, which is always a register.

====ISGN====
Integer Sign Change

===Structure and variants===
  * <code>ISGN { Register1 }</code>

===Processing actions===
  * <code>Register1 = -Register1</code>

===Description===
ISGN interprets the operand register as an integer and inverts its sign.

====IMIN====
Integer Minimum

===Structure and variants===
  * <code>(Variant 1): IMIN { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): IMIN { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 = min( Register1, ImmediateValue )</code>
  * <code>(Variant 2): Register1 = min( Register1, Register2 )</code>

===Description===
IMIN interprets both of its operands as integers. It then takes the minimum of both
values and stores it in the first operand, which is always a register.

====IMAX====
Integer Maximum

===Structure and variants===
  * <code>(Variant 1): IMAX { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): IMAX { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 = max( Register1, ImmediateValue )</code>
  * <code>(Variant 2): Register1 = max( Register1, Register2 )</code>

===Description===
IMAX interprets both of its operands as integers. It then takes the maximum of both
values and stores it in the first operand, which is always a register.

====IABS====
Integer Absolute Value

===Structure and variants===
  * <code>IABS { Register1 }</code>

===Processing actions===
  * <code>Register1 = abs( Register1 )</code>

===Description===
IABS interprets the operand register as an integer and takes its absolute value.

====FADD====
Float Addition

===Structure and variants===
  * <code>(Variant 1): FADD { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): FADD { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 += ImmediateValue</code>
  * <code>(Variant 2): Register1 += Register2</code>

===Description===
FADD interprets both of its operands as floats and performs an addition. The result is
stored in the first operand, which is always a register. Overflow bits are discarded.

====FSUB====
Float Subtraction

===Structure and variants===
  * <code>(Variant 1): FSUB { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): FSUB { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 -= ImmediateValue</code>
  * <code>(Variant 2): Register1 -= Register2</code>

===Description===
FSUB interprets both of its operands as floats and performs a subtraction. The result
is stored in the first operand, which is always a register. Overflow bits are discarded.

====FMUL====
Float Multiplication

===Structure and variants===
  * <code>(Variant 1): FMUL { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): FMUL { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 *= ImmediateValue</code>
  * <code>(Variant 2): Register1 *= Register2</code>

===Description===
FMUL interprets both of its operands as floats and performs a multiplication. The
result is stored in the first operand, which is always a register. Overflow bits are
discarded.

====FDIV====
Float Division

===Structure and variants===
  * <code>(Variant 1): FDIV { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): FDIV { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 /= ImmediateValue</code>
  * <code>(Variant 2): Register1 /= Register2</code>

===Description===
FDIV interprets both of its operands as floats and performs a division. The result is
stored in the first operand, which is always a register.

====FMOD====
Float Modulus

===Structure and variants===
  * <code>(Variant 1): FMOD { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): FMOD { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 = fmod(Register1, ImmediateValue)</code>
  * <code>(Variant 2): Register1 = fmod(Register1, Register2)</code>

===Description===
FMOD interprets both of its operands as floats and performs a division. It then takes the
remainder of that division when the result’s fractional part is discarded and stores it in
the first operand, which is always a register.

====FSGN====
Float Sign Change

===Structure and variants===
  * <code>FSGN { Register1 }</code>

===Processing actions===
  * <code>Register1 = -Register1</code>

===Description===
FSGN interprets the operand register as a float and inverts its sign.

====FMIN====
Float Minimum

===Structure and variants===
  * <code>(Variant 1): FMIN { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): FMIN { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 = min( Register1, ImmediateValue )</code>
  * <code>(Variant 2): Register1 = min( Register1, Register2 )</code>

===Description===
FMIN interprets both of its operands as floats. It then takes the minimum of both
values and stores it in the first operand, which is always a register.

====FMAX====
Float Maximum

===Structure and variants===
  * <code>(Variant 1): FMAX { Register1 }, { ImmediateValue }</code>
  * <code>(Variant 2): FMAX { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>(Variant 1): Register1 = max( Register1, ImmediateValue )</code>
  * <code>(Variant 2): Register1 = max( Register1, Register2 )</code>

===Description===
FMAX interprets both of its operands as floats. It then takes the maximum of both
values and stores it in the first operand, which is always a register.

====FABS====
Float Absolute Value

===Structure and variants===
  * <code>FABS { Register1 }</code>

===Processing actions===
  * <code>Register1 = abs( Register1 )</code>

===Description===
FABS interprets the operand register as a float and takes its absolute value.


====FLR====
Round down

===Structure and variants===
  * <code>FLR { Register1 }</code>

===Processing actions===
  * <code>Register1 = floor( Register1 )</code>

===Description===
FLR interprets the operand register as a float and rounds it downwards to an integer
value. Note that the result is not converted to an integer, but is still a float.

====CEIL====
Round up

===Structure and variants===
  * <code>CEIL { Register1 }</code>

===Processing actions===
  * <code>Register1 = ceil( Register1 )</code>

===Description===
CEIL interprets the operand register as a float and rounds it upwards to an integer
value. Note that the result is not converted to an integer, but is still a float.

====ROUND====
Round to nearest integer

===Structure and variants===
  * <code>ROUND { Register1 }</code>

===Processing actions===
  * <code>Register1 = round( Register1 )</code>

===Description===
ROUND interprets the operand register as a float and rounds it to the closest integer
value. Note that the result is not converted to an integer, but is still a float.

====SIN====
Sine

===Structure and variants===
  * <code>SIN { Register1 }</code>

===Processing actions===
  * <code>Register1 = sin( Register1 )</code>

===Description===
SIN interprets the operand register as a float and calculates the sine of that value. The
sine function will interpret its argument in radians.

====ACOS====
Arc cosine

===Structure and variants===
  * <code>ACOS { Register1 }</code>

===Processing actions===
  * <code>Register1 = acos( Register1 )</code>

===Description===
ACOS interprets the operand register as a float and calculates the arc cosine of that
value. The result is given in radians, in the range [0, pi].

====ATAN2====
Arc Tangent from x and y

===Structure and variants===
  * <code>ATAN2 { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>Register1 = atan2( Register1, Register2 )</code>

===Description===
ATAN2 interprets both operand registers as floats and calculates the angle of a vector
such that Vx = Register2 and Vy = Register1. The result is stored in the first operand
register and will be given in radians, in the range [-pi, pi]. The origin of angles is located
at (Vx > 0, Vy = 0) and angles grow when rotating towards (Vx = 0, Vy > 0).

====LOG====
Natural logarithm

===Structure and variants===
  * <code>LOG { Register1 }</code>

===Processing actions===
  * <code>Register1 = log( Register1 )</code>

===Description===
LOG interprets the operand register as a float and calculates the logarithm base e of that
value.

====POW====
Raise to a Power

===Structure and variants===
  * <code>POW { Register1 }, { Register2 }</code>

===Processing actions===
  * <code>Register1 = pow( Register1, Register2 )</code>

===Description===
POW interprets both operand registers as floats and calculates the result of raising the
first operand to the power of the second operand. The result is stored in the first operand
register.


====To Be Done====
<code>
    // -----------------------------------------------------------------------------

    enum class CPURegisters: int
    {
        // all 16 general-purpose registers
        Register00 = 0,
        Register01,
        Register02,
        Register03,
        Register04,
        Register05,
        Register06,
        Register07,
        Register08,
        Register09,
        Register10,
        Register11,
        Register12,
        Register13,
        Register14,
        Register15,

        // alternate names for specific registers
        CountRegister       = 11,
        SourceRegister      = 12,
        DestinationRegister = 13,
        BasePointer         = 14,
        StackPointer        = 15
    };

    // -----------------------------------------------------------------------------

    enum class AddressingModes : unsigned int
    {
        RegisterFromImmediate = 0,      // syntax: MOV R1, 25
        RegisterFromRegister,           // syntax: MOV R1, R2
        RegisterFromImmediateAddress,   // syntax: MOV R1, [25]
        RegisterFromRegisterAddress,    // syntax: MOV R1, [R2]
        RegisterFromAddressOffset,      // syntax: MOV R1, [R2+25]
        ImmediateAddressFromRegister,   // syntax: MOV [25], R2
        RegisterAddressFromRegister,    // syntax: MOV [R1], R2
        AddressOffsetFromRegister       // syntax: MOV [R1+25], R2
    };

    // -----------------------------------------------------------------------------

    enum class CPUErrorCodes: uint32_t
    {
        InvalidMemoryRead = 0,
        InvalidMemoryWrite,
        InvalidPortRead,
        InvalidPortWrite,
        StackOverflow,
        StackUnderflow,
        DivisionError,
        ArcCosineError,
        ArcTangent2Error,
        LogarithmError,
        PowerError
    };
}
</code>
