# Testing Instructions for MUX7x1_8 Fix

## Quick Test Guide

After implementing the MUX7x1_8 multiplexer according to the Implementation Guide, follow these steps to verify the fix works correctly.

## Test Environment Setup

### Required Tools
- LogicCircuit application
- PROYECTO1_albin-greife_gaudy-montero(1).CircuitProject (with MUX implemented)

### Test Methodology
The test verifies that the oscillation stage now correctly uses the figure pattern corresponding to the active size/side, rather than always using the size-4 pattern.

## Test Cases

### Test Case 1: Size 1 Left (1Z)
**Setup:**
1. Load the circuit in LogicCircuit
2. Set the tamaño module input to produce 1Z output (figure code that results in size 1, left position)
3. Expected one-hot state: 1Z=1, all others=0

**Verification:**
1. Check MUX output (fig_sel): Should match the 1IZ1 bus from Movimiento
2. Observe oscillation: Should use size-1 left pattern
3. **Pass Criteria:** Oscillation uses 1-block pattern on left side, NOT 4-block pattern

### Test Case 2: Size 1 Right (1D)
**Setup:**
1. Set tamaño input for size 1, right position (1D=1)

**Verification:**
1. MUX output should match 1IZ2 bus
2. Oscillation should use size-1 right pattern
3. **Pass Criteria:** Oscillation uses 1-block pattern on right side

### Test Case 3: Size 2 Left (2I)
**Setup:**
1. Set tamaño input for size 2, left position (2I=1)

**Verification:**
1. MUX output should match 2I1 bus
2. Oscillation should use size-2 left pattern
3. **Pass Criteria:** Oscillation uses 2-block pattern on left side

### Test Case 4: Size 2 Right (2D)
**Setup:**
1. Set tamaño input for size 2, right position (2D=1)

**Verification:**
1. MUX output should match 2D1 bus
2. Oscillation should use size-2 right pattern
3. **Pass Criteria:** Oscillation uses 2-block pattern on right side

### Test Case 5: Size 3 Left (3IZ)
**Setup:**
1. Set tamaño input for size 3, left position (3IZ=1)

**Verification:**
1. MUX output should match 3I1 bus
2. Oscillation should use size-3 left pattern
3. **Pass Criteria:** Oscillation uses 3-block pattern on left side

### Test Case 6: Size 3 Right (3D)
**Setup:**
1. Set tamaño input for size 3, right position (3D=1)

**Verification:**
1. MUX output should match 3D1 bus
2. Oscillation should use size-3 right pattern
3. **Pass Criteria:** Oscillation uses 3-block pattern on right side

### Test Case 7: Size 4 Full (4FULL)
**Setup:**
1. Set tamaño input for size 4 (4FULL=1)

**Verification:**
1. MUX output should match 4f bus
2. Oscillation should use size-4 full pattern
3. **Pass Criteria:** Oscillation uses 4-block pattern (this should work as before)

## Regression Tests

### Test Case 8: No Active Figure
**Setup:**
1. Set all tamaño outputs to 0 (no figure selected)

**Verification:**
1. MUX output should be all zeros
2. No oscillation or default behavior
3. **Pass Criteria:** System handles null case gracefully

### Test Case 9: Matrix Unification (m/m1m2)
**Setup:**
1. Test each figure size/side as above

**Verification:**
1. Verify that left/right matrix separation still works correctly
2. Check that the dual 8x8 matrix setup remains functional
3. **Pass Criteria:** No regression in matrix unification logic

### Test Case 10: Vertical Handling
**Setup:**
1. Test figures with vertical components

**Verification:**
1. Verify vertical oscillation still works
2. **Pass Criteria:** No regression in vertical handling logic

## Debugging Failed Tests

### Symptom: MUX output is always zero
**Possible causes:**
- Selector signals not reaching MUX
- Data buses not properly connected to MUX
- MUX internal logic error (AND gates not working)

**Debug steps:**
1. Use LogicCircuit's probe tool to check selector signals at MUX input
2. Check data bus values at MUX input
3. Step through MUX internal logic to find where signal is lost

### Symptom: Wrong figure pattern
**Possible causes:**
- Incorrect mapping between selectors and data buses
- Movimiento outputs wired to wrong MUX inputs
- MUX output connected to wrong destination

**Debug steps:**
1. Create a mapping table of current connections
2. Compare with the specification in the Implementation Guide
3. Verify each wire connection manually

### Symptom: All figures use same pattern
**Possible causes:**
- MUX not actually in the signal path
- Original hardwired connection still active
- MUX output not connected

**Debug steps:**
1. Trace signal flow from tamaño → MUX → oscillation logic
2. Check if old connection bypasses the MUX
3. Verify MUX output is actually connected to oscillation input

### Symptom: Circuit doesn't load
**Possible causes:**
- XML syntax error
- Invalid circuit references
- Duplicate GUIDs

**Debug steps:**
1. Check LogicCircuit error messages
2. Validate XML syntax
3. Search for duplicate circuit or pin IDs
4. Restore from backup and retry

## Performance Validation

### Timing
- Verify that MUX doesn't introduce unacceptable propagation delay
- Check that oscillation timing remains correct
- **Pass Criteria:** Game remains playable at original speed

### Resource Usage
- MUX uses 7 REGULADOR instances + 6 OR8BITS instances
- Total: 13 additional subcircuit instances
- **Pass Criteria:** Circuit remains within LogicCircuit's limits

## Acceptance Criteria Summary

✅ **All 7 figure size/side combinations** oscillate with their correct patterns  
✅ **Size-4 regression**: Still works as it did before (now explicitly selected)  
✅ **No regressions**: Matrix unification and vertical handling unchanged  
✅ **Performance**: No noticeable slowdown  
✅ **Code quality**: Circuit remains maintainable with clear labels  

## Test Results Template

```
Test Date: _____________
Tester: _____________

Test Case 1 (1Z): [ ] PASS [ ] FAIL
Test Case 2 (1D): [ ] PASS [ ] FAIL
Test Case 3 (2I): [ ] PASS [ ] FAIL
Test Case 4 (2D): [ ] PASS [ ] FAIL
Test Case 5 (3IZ): [ ] PASS [ ] FAIL
Test Case 6 (3D): [ ] PASS [ ] FAIL
Test Case 7 (4FULL): [ ] PASS [ ] FAIL
Test Case 8 (null): [ ] PASS [ ] FAIL
Test Case 9 (matrix): [ ] PASS [ ] FAIL
Test Case 10 (vertical): [ ] PASS [ ] FAIL

Overall Result: [ ] ALL PASS [ ] SOME FAILURES

Notes:
_______________________________________________
_______________________________________________
_______________________________________________
```

## Quick Toggle Test

For rapid testing, you can set up switches or inputs that directly toggle the 7 selector lines:

1. Add 7 input switches labeled s0 through s6
2. Wire each to the corresponding tamaño output (or MUX selector input for isolated testing)
3. Toggle each switch one at a time
4. Observe that the corresponding figure pattern is used

This allows you to quickly cycle through all 7 cases without having to generate specific figure codes.

## Visual Verification

Create a reference card showing what each figure pattern should look like:

```
1Z (1IZ1): [■]□□□□□□□  (single block, left)
1D (1IZ2): □□□□□□□[■]  (single block, right)
2I (2I1):  [■■]□□□□□□  (two blocks, left)
2D (2D1):  □□□□□□[■■]  (two blocks, right)
3IZ (3I1): [■■■]□□□□□  (three blocks, left)
3D (3D1):  □□□□□[■■■]  (three blocks, right)
4F (4f):   [■■■■■■■■]  (four blocks, full)
```

Compare the actual oscillation pattern with this reference during testing.

## Final Verification

Before considering the fix complete:

1. [ ] All test cases pass
2. [ ] No regressions identified
3. [ ] Performance is acceptable
4. [ ] Code is documented
5. [ ] Backup of working version created
6. [ ] Changes committed to version control

## Support

If you encounter issues during testing:
1. Check the Implementation Guide for wiring details
2. Review the pin ID reference section
3. Use LogicCircuit's debugging tools (probes, step execution)
4. Document any unexpected behavior for troubleshooting
