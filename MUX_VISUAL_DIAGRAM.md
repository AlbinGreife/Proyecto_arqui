# Visual Circuit Diagram for MUX7x1_8 Implementation

## Signal Flow Overview

```
┌──────────┐
│  tamaño  │  (Decodes figure size/side from 4-bit input)
└────┬─────┘
     │ 7 one-hot outputs (1-bit each)
     │ 1Z, 1D, 2I, 2D, 3IZ, 3D, 4FULL
     │
     ├─────────────────┐
     │                 │
     │                 ↓
     │            ┌────────────┐
     │            │  MUX7x1_8  │
     │            │            │
     │  s0 ←──────┤ s0_1Z      │
     │  s1 ←──────┤ s1_1D      │
     │  s2 ←──────┤ s2_2I      │
     │  s3 ←──────┤ s3_2D      │
     │  s4 ←──────┤ s4_3IZ     │
     │  s5 ←──────┤ s5_3D      │
     │  s6 ←──────┤ s6_4FULL   │
     │            │            │
     │            │ d0_1IZ1 ───┤← 1IZ1 (8-bit)
     │            │ d1_1IZ2 ───┤← 1IZ2 (8-bit)
     │            │ d2_2I1  ───┤← 2I1  (8-bit)
     │            │ d3_2D1  ───┤← 2D1  (8-bit)
     │            │ d4_3I1  ───┤← 3I1  (8-bit)
     │            │ d5_3D1  ───┤← 3D1  (8-bit)
     │            │ d6_4f   ───┤← 4f   (8-bit)
     │            │            │
     │            │   fig_sel ─┼──→ To Oscillation Logic
     │            └────────────┘    (8-bit selected figure)
     │                 ↑
     │                 │
┌────┴──────┐         │
│ Movimiento│  ───────┘
│           │  7 outputs (8-bit each)
│           │  1IZ1, 1IZ2, 2I1, 2D1, 3I1, 3D1, 4f
└───────────┘
```

## MUX7x1_8 Internal Logic

### Conceptual View
```
For each bit position i (0 to 7):
  fig_sel[i] = (d0[i] & s0) | (d1[i] & s1) | (d2[i] & s2) | 
               (d3[i] & s3) | (d4[i] & s4) | (d5[i] & s5) | (d6[i] & s6)
```

### Component-Level Implementation
```
Input: d0[7:0] ──┐
Input: s0 ────┐  │
              ↓  ↓
         ┌────────────┐
         │ REGULADOR  │  (8-bit AND: d0 & replicate(s0,8))
         │   (AND)    │
         └─────┬──────┘
               │ gated_d0[7:0]
               ↓
         ┌────────────┐
Input: d1[7:0] ──┤            │
Input: s1 ────→  │ REGULADOR  │
                 │   (AND)    │
                 └─────┬──────┘
                       │ gated_d1[7:0]
                       ↓
                 ┌────────────┐
   gated_d0 ─────┤            │
   gated_d1 ─────┤  OR8BITS   │
                 └─────┬──────┘
                       │ partial_or_01[7:0]
                       ↓
                 ┌────────────┐
Input: d2[7:0] ──┤            │
Input: s2 ────→  │ REGULADOR  │
                 │   (AND)    │
                 └─────┬──────┘
                       │ gated_d2[7:0]
                       ↓
                 ┌────────────┐
partial_or_01 ───┤            │
gated_d2 ────────┤  OR8BITS   │
                 └─────┬──────┘
                       │ partial_or_012[7:0]
                       ↓
                      ...
                       │
                 ┌────────────┐
Input: d6[7:0] ──┤            │
Input: s6 ────→  │ REGULADOR  │
                 │   (AND)    │
                 └─────┬──────┘
                       │ gated_d6[7:0]
                       ↓
                 ┌────────────┐
partial_or_...───┤            │
gated_d6 ────────┤  OR8BITS   │
                 └─────┬──────┘
                       │
                       ↓
                  fig_sel[7:0] (OUTPUT)
```

## Simplified Single-Bit View

For better understanding, here's how ONE bit (e.g., bit 0) flows through the MUX:

```
d0[0] ──┬──AND──s0──┐
                    │
d1[0] ──┬──AND──s1──┤
                    │
d2[0] ──┬──AND──s2──┤
                    ├──OR────→ fig_sel[0]
d3[0] ──┬──AND──s3──┤
                    │
d4[0] ──┬──AND──s4──┤
                    │
d5[0] ──┬──AND──s5──┤
                    │
d6[0] ──┬──AND──s6──┘

(This pattern repeats for bits 1-7)
```

## Selector Truth Table

Only ONE selector should be active (1) at a time (one-hot encoding):

| s0 | s1 | s2 | s3 | s4 | s5 | s6 | Output (fig_sel) |
|----|----|----|----|----|----|----|-------------------|
| 1  | 0  | 0  | 0  | 0  | 0  | 0  | d0 (1IZ1)        |
| 0  | 1  | 0  | 0  | 0  | 0  | 0  | d1 (1IZ2)        |
| 0  | 0  | 1  | 0  | 0  | 0  | 0  | d2 (2I1)         |
| 0  | 0  | 0  | 1  | 0  | 0  | 0  | d3 (2D1)         |
| 0  | 0  | 0  | 0  | 1  | 0  | 0  | d4 (3I1)         |
| 0  | 0  | 0  | 0  | 0  | 1  | 0  | d5 (3D1)         |
| 0  | 0  | 0  | 0  | 0  | 0  | 1  | d6 (4f)          |
| 0  | 0  | 0  | 0  | 0  | 0  | 0  | 0x00 (no figure) |

## Component Placement Strategy in LogicCircuit

### Layout Suggestion
```
Left Side (Inputs):
  Top:    Data inputs (d0-d6), stacked vertically
  Bottom: Select inputs (s0-s6), stacked vertically

Center (Logic):
  7 REGULADOR blocks, aligned vertically
  6 OR8BITS blocks, arranged in a combining tree or chain

Right Side (Output):
  fig_sel output pin
```

### Example Coordinates (adjust as needed)
```
Data Input Pins:
  d0: X=10, Y=10
  d1: X=10, Y=20
  d2: X=10, Y=30
  d3: X=10, Y=40
  d4: X=10, Y=50
  d5: X=10, Y=60
  d6: X=10, Y=70

Select Input Pins:
  s0: X=10, Y=90
  s1: X=10, Y=95
  s2: X=10, Y=100
  s3: X=10, Y=105
  s4: X=10, Y=110
  s5: X=10, Y=115
  s6: X=10, Y=120

REGULADOR Blocks (AND):
  REGULADOR_0: X=40, Y=15  (gates d0 with s0)
  REGULADOR_1: X=40, Y=25  (gates d1 with s1)
  REGULADOR_2: X=40, Y=35  (gates d2 with s2)
  REGULADOR_3: X=40, Y=45  (gates d3 with s3)
  REGULADOR_4: X=40, Y=55  (gates d4 with s4)
  REGULADOR_5: X=40, Y=65  (gates d5 with s5)
  REGULADOR_6: X=40, Y=75  (gates d6 with s6)

OR8BITS Blocks:
  OR_01:    X=70, Y=20  (combines gated_d0 and gated_d1)
  OR_012:   X=90, Y=30  (combines OR_01 and gated_d2)
  OR_0123:  X=110, Y=40 (combines OR_012 and gated_d3)
  OR_01234: X=130, Y=50 (combines OR_0123 and gated_d4)
  OR_012345: X=150, Y=60 (combines OR_01234 and gated_d5)
  OR_final: X=170, Y=70 (combines OR_012345 and gated_d6)

Output Pin:
  fig_sel: X=200, Y=70
```

## Wiring at FILA15 Level

```
FILA15 Circuit:
┌────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ┌──────────┐                                                   │
│  │  tamaño  │                                                   │
│  │          │  1Z ────┐                                         │
│  │          │  1D ────┤                                         │
│  │          │  2I ────┤                                         │
│  │          │  2D ────┼───→ (Connect to MUX select inputs)     │
│  │          │  3IZ ───┤                                         │
│  │          │  3D ────┤                                         │
│  │          │  4FULL ─┘                                         │
│  └──────────┘                                                   │
│                                                                  │
│  ┌────────────┐                                                 │
│  │ Movimiento │                                                 │
│  │            │  1IZ1 ─┐                                        │
│  │            │  1IZ2 ─┤                                        │
│  │            │  2I1  ─┤                                        │
│  │            │  2D1  ─┼───→ (Connect to MUX data inputs)      │
│  │            │  3I1  ─┤                                        │
│  │            │  3D1  ─┤                                        │
│  │            │  4f   ─┘                                        │
│  └────────────┘                                                 │
│                         ┌────────────┐                          │
│                         │  MUX7x1_8  │                          │
│                         │            │                          │
│  (selectors) ──────────→│ s0..s6     │                          │
│  (data buses) ─────────→│ d0..d6     │                          │
│                         │            │                          │
│                         │   fig_sel ─┼──→ To q1 (output)       │
│                         └────────────┘    or to oscillation     │
│                                                                  │
└────────────────────────────────────────────────────────────────┘
```

## Quick Reference: Signal Names

### In Movimiento (outputs):
- 1IZ1, 1IZ2, 2I1, 2D1, 3I1, 3D1, 4f (all 8-bit)

### In tamaño (outputs):
- 1Z, 1D, 2I, 2D, 3IZ, 3D, 4FULL (all 1-bit)

### In MUX7x1_8:
**Data inputs (8-bit):**
- d0_1IZ1, d1_1IZ2, d2_2I1, d3_2D1, d4_3I1, d5_3D1, d6_4f

**Select inputs (1-bit):**
- s0_1Z, s1_1D, s2_2I, s3_2D, s4_3IZ, s5_3D, s6_4FULL

**Output (8-bit):**
- fig_sel

## Testing Checkpoint Locations

When testing, probe these points:

1. **tamaño outputs**: Verify one-hot encoding (only one high at a time)
2. **Movimiento outputs**: Verify correct patterns for each size/side
3. **MUX select inputs**: Verify selectors reach the MUX correctly
4. **MUX data inputs**: Verify data buses reach the MUX correctly
5. **MUX internal (gated outputs)**: Only the selected data should pass through
6. **MUX output (fig_sel)**: Should match the selected Movimiento output
7. **Final oscillation**: Should use the correct figure pattern

## Color Coding Suggestion (for clarity in LogicCircuit)

If LogicCircuit supports wire coloring or labeling:
- **Red**: Select signals (1-bit, one-hot)
- **Blue**: Data buses (8-bit figure patterns)
- **Green**: MUX output (selected 8-bit figure)
- **Yellow**: Clock and control signals

## Common Pitfalls to Avoid

1. ❌ **Swapped mappings**: Ensure s0→d0, s1→d1, etc. (not s0→d6)
2. ❌ **Missing replicate**: Select bits must be replicated to 8 bits for AND
3. ❌ **Wrong OR chain**: Must combine all 7 gated outputs, not just some
4. ❌ **Bypassed MUX**: Ensure old direct connection is removed
5. ❌ **Unconnected pins**: All MUX pins must be wired
6. ❌ **Wrong circuit level**: MUX must be where both tamaño and Movimiento outputs are available

## Success Indicators

✅ Each selector activates its corresponding data input  
✅ Non-selected inputs are gated to zero  
✅ MUX output equals the selected input  
✅ Oscillation pattern matches the active figure size/side  
✅ No signal integrity issues (no floating or conflicting values)  
