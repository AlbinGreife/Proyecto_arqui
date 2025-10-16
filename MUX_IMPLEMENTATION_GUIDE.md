# MUX7x1_8 Implementation Guide

## Overview
This document provides detailed instructions for implementing a 7-to-1 8-bit multiplexer (MUX7x1_8) to fix the oscillation stage bug in the LogicCircuit project.

## Problem Summary
The oscillation stage currently behaves as if the figure were always size 4, regardless of the actual captured figure size (1, 2, 3, or 4) and side (izquierda/derecha). This is because the circuit uses only one specific output from the Movimiento module (the size-4 bus) instead of dynamically selecting the correct one based on the active figure.

## Solution
Create a 7→1 8-bit multiplexer that:
- Takes the 7 output buses from Movimiento (8 bits each)
- Takes the 7 one-hot selector outputs from tamaño (1 bit each)
- Outputs the selected 8-bit bus to feed into the oscillation logic

## Step-by-Step Implementation

### Step 1: Open the Project in LogicCircuit
1. Launch the LogicCircuit application
2. Open: `PROYECTO1_albin-greife_gaudy-montero(1).CircuitProject`

### Step 2: Create the MUX7x1_8 Subcircuit

#### 2.1 Create New Circuit
1. Right-click in the circuit list panel
2. Select "New Circuit"
3. Name it: **MUX7x1_8**
4. Set Notation to: **MUX7x1_8**
5. Add Note: "7-to-1 8-bit multiplexer for figure selection"
6. Set Category: **Operadores 8 bits**

#### 2.2 Add Input Pins (Data)
Add 7 input pins for the 8-bit data buses:
| Pin Name | BitWidth | JamNotation | Description |
|----------|----------|-------------|-------------|
| d0       | 8        | D0_1IZ1     | Figure 1 Left (1IZ1) |
| d1       | 8        | D1_1IZ2     | Figure 1 Right (1IZ2) |
| d2       | 8        | D2_2I1      | Figure 2 Left (2I1) |
| d3       | 8        | D3_2D1      | Figure 2 Right (2D1) |
| d4       | 8        | D4_3I1      | Figure 3 Left (3I1) |
| d5       | 8        | D5_3D1      | Figure 3 Right (3D1) |
| d6       | 8        | D6_4F       | Figure 4 Full (4f) |

#### 2.3 Add Input Pins (Select)
Add 7 input pins for the 1-bit one-hot selectors:
| Pin Name | BitWidth | JamNotation | Description |
|----------|----------|-------------|-------------|
| s0       | 1        | S0_1Z       | Select Figure 1 Left |
| s1       | 1        | S1_1D       | Select Figure 1 Right |
| s2       | 1        | S2_2I       | Select Figure 2 Left |
| s3       | 1        | S3_2D       | Select Figure 2 Right |
| s4       | 1        | S4_3IZ      | Select Figure 3 Left |
| s5       | 1        | S5_3D       | Select Figure 3 Right |
| s6       | 1        | S6_4FULL    | Select Figure 4 Full |

#### 2.4 Add Output Pin
| Pin Name | BitWidth | JamNotation | PinSide | Description |
|----------|----------|-------------|---------|-------------|
| fig_sel  | 8        | FIG_SEL     | Right   | Selected figure bus |

### Step 3: Build the MUX Logic

The multiplexer logic works as follows:
```
fig_sel = (d0 & replicate(s0, 8)) | 
          (d1 & replicate(s1, 8)) |
          (d2 & replicate(s2, 8)) |
          (d3 & replicate(s3, 8)) |
          (d4 & replicate(s4, 8)) |
          (d5 & replicate(s5, 8)) |
          (d6 & replicate(s6, 8))
```

#### 3.1 Add Components for Each Data Input (Repeat 7 times)

For each data input (d0 through d6), add:

1. **Splitter** (to split the 1-bit selector into 8 bits):
   - Place a Splitter component
   - Set input to the selector pin (s0, s1, ..., s6)
   - Set output to 8 separate 1-bit signals
   - Or use a simpler method: directly connect the selector bit to all 8 AND gates if the REGULADOR accepts this

2. **REGULADOR** (8-bit AND):
   - Place one REGULADOR component from "Operadores 8 bits"
   - Connect the 8-bit data input (d0, d1, ..., d6) to one side
   - Connect the replicated selector signal to the other side
   - This gates the data bus: outputs all 0s if selector is 0, passes data if selector is 1

#### 3.2 Combine All Gated Outputs

Use a chain of **OR8BITS** components:
1. Place 6 OR8BITS components
2. Connect the first OR8BITS to the outputs of REGULADOR #0 and #1
3. Connect the second OR8BITS to the output of the first OR8BITS and REGULADOR #2
4. Continue chaining until all 7 gated buses are combined
5. Connect the final OR8BITS output to the fig_sel output pin

**Alternative simpler approach**: If LogicCircuit supports multi-input OR blocks or allows chaining, use fewer OR8BITS by combining pairs first, then combining the results in a tree structure.

### Step 4: Wire the MUX in the System

#### 4.1 Identify Where to Place the MUX
The MUX should be placed where:
- Movimiento's 7 output buses are available
- tamaño's 7 one-hot selectors are available
- The combined output needs to feed into the oscillation logic

Based on analysis, this is likely in:
- **FILA15** circuit, or
- **General** circuit

#### 4.2 Add MUX Instance
1. Open the target circuit (FILA15 or General)
2. Place an instance of MUX7x1_8
3. Position it appropriately between:
   - Movimiento outputs (left side)
   - tamaño outputs (left side)
   - Oscillation logic input (right side)

#### 4.3 Wire Data Inputs
Connect the 7 Movimiento output buses to the MUX data inputs:
| Movimiento Output | MUX Data Input | Mapping |
|-------------------|----------------|---------|
| 1IZ1 (8-bit)      | d0             | 1Z → 1IZ1 |
| 1IZ2 (8-bit)      | d1             | 1D → 1IZ2 |
| 2I1 (8-bit)       | d2             | 2I → 2I1 |
| 2D1 (8-bit)       | d3             | 2D → 2D1 |
| 3I1 (8-bit)       | d4             | 3IZ → 3I1 |
| 3D1 (8-bit)       | d5             | 3D → 3D1 |
| 4f (8-bit)        | d6             | 4FULL → 4f |

#### 4.4 Wire Select Inputs
Connect the 7 tamaño output signals to the MUX select inputs:
| tamaño Output | MUX Select Input |
|---------------|------------------|
| 1Z            | s0               |
| 1D            | s1               |
| 2I            | s2               |
| 2D            | s3               |
| 3IZ           | s4               |
| 3D            | s5               |
| 4FULL         | s6               |

#### 4.5 Wire MUX Output
1. Identify the current connection that uses only the 4f output (the one causing the bug)
2. Remove or reroute that connection
3. Connect the MUX fig_sel output to the oscillation logic input instead

**Specific location to check**:
- In FILA15: The output pin q1 appears to come from Movimiento
- Trace where this q1 is currently wired and replace with MUX output

### Step 5: Test the Fix

#### 5.1 Test Each Figure Size/Side
For each combination, verify that the oscillation uses the correct figure:
1. 1Z (size 1, left): Should oscillate with 1IZ1 pattern
2. 1D (size 1, right): Should oscillate with 1IZ2 pattern
3. 2I (size 2, left): Should oscillate with 2I1 pattern
4. 2D (size 2, right): Should oscillate with 2D1 pattern
5. 3IZ (size 3, left): Should oscillate with 3I1 pattern
6. 3D (size 3, right): Should oscillate with 3D1 pattern
7. 4FULL (size 4): Should oscillate with 4f pattern

#### 5.2 Verify One-Hot Behavior
Ensure that:
- Only one selector is active at a time (tamaño's one-hot property)
- The MUX outputs all zeros if no selector is active
- The MUX outputs the correct bus when a selector is active

### Step 6: Document Changes
1. Add comments or notes to the MUX7x1_8 circuit explaining its purpose
2. Update any project documentation
3. Note the mapping between selectors and data inputs

## Changes Already Made

### Pin Annotation Update
✅ The "D O I?" input pin x annotation has been changed from "4f" to "fig" to avoid confusion.
- **File**: PROYECTO1_albin-greife_gaudy-montero(1).CircuitProject
- **Change**: Pin f1169cd7-2ee3-4fa6-bb69-8ee40f4b2e47 JamNotation: "4f" → "fig"
- **Purpose**: Remove misleading hardcoded reference to size-4 bus

## Notes

### Why This Fix Works
The current implementation has 7 "D O I?" instances inside Movimiento, each processing a different figure size/side. However, somewhere in the circuit (likely at the FILA15 output), only ONE of these processed outputs is being used - specifically the one for size 4. The MUX allows dynamic selection of the correct processed output based on the currently active figure.

### One-Hot Encoding
The tamaño module produces mutually exclusive one-hot signals, meaning exactly one is 1 and the rest are 0 (or all 0 if no figure is active). This ensures the MUX selects exactly one input at a time.

### Alternative: OR Instead of MUX
Since the selectors are one-hot and gate the data buses, you could simply OR all 7 gated buses together (the inactive ones will be all 0s). This is essentially what the MUX does, but making it a reusable MUX component is cleaner and more maintainable.

## Troubleshooting

### MUX Always Outputs Zero
- Check that selector signals are actually reaching the MUX
- Verify tamaño is producing one-hot outputs correctly
- Check that the selector-to-data mapping is correct

### Wrong Figure Pattern
- Verify the mapping between selectors and data buses
- Check that Movimiento outputs are correctly wired to MUX inputs
- Ensure the MUX output is wired to the right destination

### Circuit Won't Load
- Check for syntax errors in the XML
- Verify all GUIDs are unique
- Make sure all circuit references are valid

## References

### Pin IDs (for reference)
**Movimiento outputs:**
- 4f: d573d2e2-1107-4557-901c-ba7e45fa2529
- 1IZ1: 61080a56-584a-4639-aa65-ad6cf2126259
- 1IZ2: 8f1c7b68-3a21-4c70-b51e-4ced30980b25
- 2I1: ae56db7b-37f0-4882-a80f-8e0373f4a37e
- 2D1: 3a43a922-595c-4d2d-ab12-9c843ba6d0fa
- 3I1: d5c4e93d-634d-4ebb-937f-180312d08424
- 3D1: 5b5a91f8-e114-451f-ac5c-92261a62b110

**tamaño outputs:**
- 1Z: a2d299bb-71b4-4553-96c3-3597c6fa4aa6
- 1D: 0d8ef5f5-b293-49b9-a1b1-f0b4a7aaf0d1
- 2I: 72891907-b0a8-4801-a01e-38dee9fabd65
- 2D: a39712a0-eb75-479c-9abc-f1f3cb2d80f2
- 3IZ: bdae954e-8abe-4bb3-a2c3-d2d3b0dcfda5
- 3D: 73748347-8fb2-4956-ac61-4e2adffadd55
- 4FULL: a7165453-6ff1-48b4-b0d0-bf1a546ed507

### Circuit IDs
- Movimiento: da2f37f6-ef23-407d-9f43-61673ca60bcb
- tamaño: 991a34f0-13d6-4222-8ef7-3521db8567d1
- D O I?: d77be8d5-a675-4ec3-8d6a-75f5854c3a35
- FILA15: 4d658cbc-49a5-43a4-864e-7037ef4152f0
- General: 7c42de23-348b-4277-8cc1-4d0517125063
- REGULADOR (8-bit AND): 6a488674-7624-4ce6-9ea8-883209f2512d
- OR8BITS: b1454e78-59c4-4808-acde-2d54a3f5bcf4
