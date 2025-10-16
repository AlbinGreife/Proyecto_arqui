# Oscillation Stage Bug Fix - MUX7x1_8 Implementation

## Problem Statement

The LogicCircuit game project has a bug where the oscillation stage always behaves as if the figure were size 4, regardless of the actual captured figure size (1, 2, 3, or 4) and side (izquierda/derecha).

## Root Cause

The circuit is configured such that the oscillation logic receives only one specific output from the Movimiento module - the size-4 figure bus (4f). The Movimiento module correctly processes all 7 figure size/side combinations internally, but the downstream logic doesn't dynamically select which one to use based on the currently active figure.

## Solution

Implement a 7-to-1 8-bit multiplexer (MUX7x1_8) that:
- Accepts 7 data inputs: the 8-bit figure buses from Movimiento (1IZ1, 1IZ2, 2I1, 2D1, 3I1, 3D1, 4f)
- Accepts 7 select inputs: the one-hot size/side signals from tamaño (1Z, 1D, 2I, 2D, 3IZ, 3D, 4FULL)
- Outputs the selected 8-bit figure bus to feed into the oscillation logic

## What's Included

### Documentation
1. **MUX_IMPLEMENTATION_GUIDE.md** - Complete step-by-step instructions for creating the MUX subcircuit in LogicCircuit
2. **MUX_TESTING_GUIDE.md** - Comprehensive testing procedures to verify the fix works correctly
3. **README.md** - This file, providing an overview of the fix

### Code Changes
1. **Pin Annotation Update** - Changed the "D O I?" input pin x JamNotation from "4f" to "fig" to remove the misleading hardcoded reference

## Implementation Status

### ✅ Completed
- [x] Analyzed the CircuitProject XML structure
- [x] Identified the bug location and root cause
- [x] Mapped all relevant pins and circuit IDs
- [x] Updated misleading pin annotation
- [x] Created comprehensive implementation guide
- [x] Created testing guide

### ⚠️ Requires Manual Completion in LogicCircuit GUI
Due to the complexity of the LogicCircuit XML format and the need for precise visual layout, the following steps must be completed manually in the LogicCircuit application:

- [ ] Create MUX7x1_8 subcircuit definition
- [ ] Build MUX internal logic using REGULADOR and OR8BITS blocks
- [ ] Instantiate MUX in the appropriate circuit (likely FILA15)
- [ ] Wire Movimiento outputs to MUX data inputs
- [ ] Wire tamaño outputs to MUX select inputs
- [ ] Wire MUX output to oscillation logic
- [ ] Test all 7 figure size/side combinations

## Why Manual Completion is Necessary

The LogicCircuit XML format includes:
- Circuit definitions and hierarchical relationships
- Component placements with precise X,Y coordinates
- Wire routing with coordinate-based connections
- Internal circuit logic and gate placements

Creating all of this correctly by hand-editing XML would be:
- Extremely error-prone
- Time-consuming to debug
- Likely to corrupt the circuit file
- Impossible to verify without the GUI

The LogicCircuit GUI provides:
- Visual circuit editing
- Automatic wire routing
- Component snapping and alignment
- Real-time validation
- Simulation and testing tools

## How to Proceed

1. **Open the project**: Load `PROYECTO1_albin-greife_gaudy-montero(1).CircuitProject` in LogicCircuit
2. **Follow the Implementation Guide**: See `MUX_IMPLEMENTATION_GUIDE.md` for detailed step-by-step instructions
3. **Test thoroughly**: Use `MUX_TESTING_GUIDE.md` to verify the fix works for all 7 figure combinations
4. **Verify no regressions**: Ensure matrix unification and vertical handling still work correctly

## Signal Mapping Reference

### Data Input Mapping (Movimiento → MUX)
| Movimiento Output | MUX Data Input | Description |
|-------------------|----------------|-------------|
| 1IZ1              | d0             | Size 1 Left |
| 1IZ2              | d1             | Size 1 Right |
| 2I1               | d2             | Size 2 Left |
| 2D1               | d3             | Size 2 Right |
| 3I1               | d4             | Size 3 Left |
| 3D1               | d5             | Size 3 Right |
| 4f                | d6             | Size 4 Full |

### Select Input Mapping (tamaño → MUX)
| tamaño Output | MUX Select Input | Description |
|---------------|------------------|-------------|
| 1Z            | s0               | Select Size 1 Left |
| 1D            | s1               | Select Size 1 Right |
| 2I            | s2               | Select Size 2 Left |
| 2D            | s3               | Select Size 2 Right |
| 3IZ           | s4               | Select Size 3 Left |
| 3D            | s5               | Select Size 3 Right |
| 4FULL         | s6               | Select Size 4 Full |

## Expected Results

After implementing the fix:
- ✅ Size-1 figures (left and right) will oscillate with their correct 1-block pattern
- ✅ Size-2 figures (left and right) will oscillate with their correct 2-block pattern
- ✅ Size-3 figures (left and right) will oscillate with their correct 3-block pattern
- ✅ Size-4 figures will continue to oscillate with the 4-block pattern (as before)
- ✅ No regressions in other circuit functionality

## Files in This Package

```
PROYECTO1_albin-greife_gaudy-montero(1).CircuitProject
├── (Modified: "D O I?" pin annotation changed from "4f" to "fig")
├── PROYECTO1_albin-greife_gaudy-montero(1).CircuitProject.backup
│   └── (Original file backup)
├── README.md
│   └── (This file: Overview and summary)
├── MUX_IMPLEMENTATION_GUIDE.md
│   └── (Step-by-step instructions for implementing the MUX)
└── MUX_TESTING_GUIDE.md
    └── (Testing procedures and validation)
```

## Quick Start

```bash
# 1. Review the changes
cat README.md

# 2. Open the project in LogicCircuit
# (Use the LogicCircuit application GUI)

# 3. Follow the implementation guide
cat MUX_IMPLEMENTATION_GUIDE.md

# 4. Test your implementation
cat MUX_TESTING_GUIDE.md
```

## Technical Details

### MUX Logic
The multiplexer uses a standard gate-based implementation:
```
For each bit i in [0..7]:
    fig_sel[i] = (d0[i] AND s0) OR 
                 (d1[i] AND s1) OR 
                 (d2[i] AND s2) OR 
                 (d3[i] AND s3) OR 
                 (d4[i] AND s4) OR 
                 (d5[i] AND s5) OR 
                 (d6[i] AND s6)
```

### One-Hot Encoding
The tamaño module produces one-hot encoded outputs, meaning exactly one selector is 1 while all others are 0. This ensures the MUX selects exactly one data input at a time.

### Resource Usage
- 7 × REGULADOR (8-bit AND) blocks
- 6 × OR8BITS blocks (for combining 7 gated outputs)
- Total: 13 subcircuit instances

## Support and Troubleshooting

If you encounter issues:
1. Check the Implementation Guide for detailed wiring instructions
2. Review the Testing Guide for debugging procedures
3. Verify all pin IDs and circuit IDs match the reference
4. Use LogicCircuit's built-in debugging tools (probes, step execution)

## Version History

### v1.0 (Current)
- Initial analysis and documentation
- Pin annotation update
- Implementation and testing guides created

## Contributors

- Analysis and documentation: GitHub Copilot
- Original circuit design: Albin Greife, Gaudy Montero

## License

This fix follows the same license as the original project.
