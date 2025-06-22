# Chart.js Hover Detection Issue - "Integrierte Zeitleiste" Graph

## Problem Description

The "Integrierte Zeitleiste" (Integrated Timeline) graph combines two financial phases:
- **Ansparphase** (Accumulation Phase) - Green area, years 1-25
- **Entnahmephase** (Withdrawal Phase) - Red area, years 26-50

### The Issue
When hovering over the graph, the tooltip was incorrectly displaying phase information:
- ✅ **Left side (green area)**: Correctly showed "Ansparphase"
- ❌ **Right side (red area)**: Incorrectly showed "Ansparphase" instead of "Entnahmephase"

## Root Cause Analysis

### Initial Hypothesis
Initially, we suspected the issue was with the tooltip logic that determines which phase to display based on the year value.

### Debugging Process

#### Step 1: Year-based Logic Debugging
Added console logging to see what year values were being detected:
```javascript
console.log(`Hover Debug: year=${year}, transitionYear=${transitionYear}, year > transitionYear = ${year > transitionYear}`);
```

**Discovery**: Even when hovering over the red area (expected years 26-50), the detected years were always < 25.

#### Step 2: Data Structure Investigation
Added debugging to examine the chart data arrays:
```javascript
console.log('DEBUG: withdrawalYearsWithTransition:', withdrawalYearsWithTransition.slice(0, 10));
console.log('DEBUG: accumulationYears:', accumulationYears.slice(0, 5));
```

**Discovery**: The data was correctly structured:
- Accumulation years: [0, 1, 2, 3, 4, ..., 25]
- Withdrawal years: [25, 26, 27, 28, 29, 30, ..., 50]

#### Step 3: Chart.js Interaction Mode Analysis
The real issue was identified in the Chart.js configuration:

```javascript
interaction: {
    intersect: false,
    mode: 'index'  // ← This was the problem!
}
```

## The Real Problem: Chart.js Interaction Mode

### What `mode: 'index'` does:
- Shows tooltips for **all datasets at the same x-coordinate**
- When hovering anywhere on the chart, it displays data from all 4 datasets simultaneously
- Chart.js prioritizes the first datasets in the array (Ansparphase datasets)
- The tooltip logic receives context for all datasets, not just the one being visually hovered over

### Why this caused the issue:
1. User hovers over red area (Entnahmephase visual area)
2. Chart.js detects the closest x-coordinate 
3. Due to `mode: 'index'`, it returns data from ALL datasets at that x-coordinate
4. The first dataset in the context array is always from Ansparphase
5. `context[0].parsed.x` returns a year from the Ansparphase dataset (< 25)
6. Tooltip logic: `year <= 25` → Shows "Ansparphase" ❌

## Solution

### The Fix
Changed the Chart.js interaction mode from `'index'` to `'nearest'`:

```javascript
interaction: {
    intersect: false,
    mode: 'nearest'  // ← Changed from 'index' to 'nearest'
}
```

### What `mode: 'nearest'` does:
- Detects the **closest data point** to the mouse cursor
- Returns context for the specific dataset being hovered over
- Provides accurate x-coordinate (year) values based on which visual area is actually being hovered

### Result:
- ✅ **Left side (green area)**: Shows "Jahr X (Ansparphase)"
- ✅ **Right side (red area)**: Shows "Jahr X (Entnahmephase)"
- ✅ **Transition point**: Shows special transition message

## Technical Details

### Chart Structure
The integrated timeline contains 4 datasets:
1. **Ansparphase (Nominal)** - Green filled area
2. **Ansparphase (Real)** - Green dashed line
3. **Entnahmephase (Nominal)** - Red filled area  
4. **Entnahmephase (Real)** - Red dashed line

### Data Mapping
```javascript
// Accumulation phase: years 0-25
accumulationYears = [0, 1, 2, ..., 25]

// Withdrawal phase: years 25-50 (with transition point)
withdrawalYearsWithTransition = [25, 26, 27, ..., 50]
```

### Tooltip Logic
```javascript
const phase = year > transitionYear ? 'Entnahmephase' : 'Ansparphase';
```

## Key Learnings

### Chart.js Interaction Modes
- **`mode: 'index'`**: Good for showing all data at the same x-coordinate (e.g., comparing multiple series)
- **`mode: 'nearest'`**: Better for detecting which specific visual element is being hovered over

### When to use each mode:
- **Use `'index'`** when you want to compare values across all datasets at the same point
- **Use `'nearest'`** when you need to detect which specific dataset/visual area is being interacted with

### Debugging Strategy
1. **Start with the symptom**: Incorrect tooltip display
2. **Check the data**: Verify data structure is correct
3. **Examine the interaction**: Understand how Chart.js handles mouse events
4. **Test the solution**: Verify the fix works for all scenarios

## Files Modified
- `public/etf_savings.html` - Line ~4337: Changed interaction mode from 'index' to 'nearest'

## Testing Checklist
- [x] Hover over left side (green area) shows "Ansparphase"
- [x] Hover over right side (red area) shows "Entnahmephase"  
- [x] Hover over transition point (year 25) shows transition message
- [x] All dataset values display correctly in tooltip
- [x] Smooth hover animations work properly 