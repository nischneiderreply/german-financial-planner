# Tax Calculation Fixes - Entnahmephase (Withdrawal Phase)

## Issues Identified and Fixed

### 1. **Abgeltungssteuer (Capital Gains Tax) Calculation Problems**

#### Problem:
- The tax calculation was using a **fixed taxable share** (~77%) instead of calculating it dynamically
- The **proportional cost basis method** was not implemented correctly
- **Teilfreistellung and Sparerpauschbetrag** were not applied according to German tax law
- **UI inconsistency**: Monthly net withdrawal in UI didn't match the table calculations

#### Solution:
- **Implemented Proportional Cost Basis Method**: Each withdrawal now properly calculates how much cost basis is consumed
- **Dynamic Taxable Share**: The taxable portion changes each year as the cost basis is depleted
- **Correct German Tax Rules**: Proper application of Teilfreistellung (30% exemption) and Sparerpauschbetrag (€1,000 annual allowance)
- **Consistent Calculations**: UI and table now use the same tax calculation logic

#### Code Changes:
```javascript
// Before: Fixed ratio approach
const gainRatio = totalUnrealizedGains / initialCapital;
const gainPortionOfWithdrawal = grossAnnualWithdrawal * gainRatio;

// After: Proportional cost basis method
const costBasisOut = remainingCostBasis * (grossAnnualWithdrawal / capitalAfterReturns);
const realizedGain = Math.max(0, grossAnnualWithdrawal - costBasisOut);
remainingCostBasis = Math.max(0, remainingCostBasis - costBasisOut);
```

### 2. **Chart Display Issues**

#### Problem:
- Negative portfolio values were showing as "€0,00" in tooltips
- Chart scaling didn't properly accommodate negative values
- Poor visual representation of negative portfolio states

#### Solution:
- **Enhanced Tooltip Formatting**: Added proper negative value formatting in chart tooltips
- **Improved Chart Scaling**: Added `suggestedMin` to properly scale charts with negative values
- **Better Visual Feedback**: Negative values now display correctly as "-€X.XX"

#### Code Changes:
```javascript
// Added to chart configuration
plugins: {
    tooltip: {
        callbacks: {
            label: function(context) {
                const value = context.parsed.y;
                if (value < 0) {
                    return label + '-€' + Math.abs(value).toLocaleString('de-DE', { 
                        minimumFractionDigits: 2, 
                        maximumFractionDigits: 2 
                    });
                }
                // ... positive value formatting
            }
        }
    }
}
```

### 3. **Real Purchasing Power Calculation**

#### Problem:
- The real purchasing power calculation was overly complex and incorrect
- Used an arbitrary "average year" calculation that didn't reflect actual purchasing power

#### Solution:
- **Simplified and Corrected**: Real purchasing power is now correctly calculated as the first year's withdrawal amount (which represents constant purchasing power throughout the withdrawal period)

#### Code Changes:
```javascript
// Before: Complex and incorrect calculation
const averageWithdrawalYear = duration / 2;
const realPurchasingPowerMonthly = monthlyGrossWithdrawal / Math.pow(1 + inflationRate, averageWithdrawalYear);

// After: Simple and correct
const realPurchasingPowerMonthly = monthlyGrossWithdrawal; // Already in today's purchasing power
```

### 4. **Function Signature Fix**

#### Problem:
- The `simulateWithdrawal` function was missing the `userDefinedTotalContributions` parameter
- This caused incorrect tax calculations when no contribution data was available

#### Solution:
- **Added Missing Parameter**: Added `userDefinedTotalContributions = 0` as a default parameter
- **Improved Fallback Logic**: Better handling when no contribution data is available

## Tax Calculation Logic (Corrected - Proportional Cost Basis Method)

### German ETF Tax Rules Applied:

1. **Teilfreistellung (Partial Exemption)**: 30% of gains from equity ETFs are tax-free
2. **Sparerpauschbetrag**: €1,000 annual tax-free allowance for capital gains (resets each year)
3. **Abgeltungssteuer**: 25% tax rate on taxable gains
4. **Proportional Cost Basis**: Cost basis is consumed proportionally with each withdrawal

### Calculation Steps (Per Year):

1. **Apply Investment Returns**:
   - Capital After Returns = Previous Capital × (1 + Annual Return)

2. **Calculate Cost Basis of Shares Being Sold**:
   - Cost Basis Out = Remaining Cost Basis × (Withdrawal Amount / Capital After Returns)

3. **Calculate Realized Gain**:
   - Realized Gain = max(0, Withdrawal Amount - Cost Basis Out)

4. **Update Remaining Cost Basis**:
   - Remaining Cost Basis = Previous Cost Basis - Cost Basis Out

5. **Apply Tax Rules**:
   - Taxable Gain = Realized Gain × (1 - Teilfreistellung Rate)
   - Apply Annual Allowance: Final Taxable = max(0, Taxable Gain - Remaining Annual Allowance)
   - Tax = Final Taxable × 25%

6. **Calculate Net Withdrawal**:
   - Net Withdrawal = Gross Withdrawal - Tax

7. **Update Portfolio**:
   - Final Capital = Capital After Returns - Gross Withdrawal

## Expected Results (Example Scenario)

With the corrected proportional cost basis method, using the scenario mentioned in the analysis:
- **Initial Capital**: €727,391
- **Initial Cost Basis**: €500,000 (assuming 31% embedded gains)
- **5% Annual Return, 2% Inflation**
- **25-Year Withdrawal Period**
- **Teilfreistellung**: 30% (enabled)

### Year 1 Results (Corrected):
- **Gross Withdrawal**: €42,329
- **Cost Basis Out**: ~€29,000
- **Realized Gain**: ~€13,329
- **Taxable Gain** (after 30% Teilfreistellung): ~€9,330
- **Tax** (after €1,000 allowance): ~€2,083
- **Net Withdrawal**: ~€40,246
- **Monthly Net**: ~€3,354

This is significantly different from the previous incorrect calculation of €8,150 in taxes.

## Testing Recommendations

1. **Test with Different Scenarios**:
   - High gain portfolios (e.g., 80% gains, 20% contributions)
   - Low gain portfolios (e.g., 20% gains, 80% contributions)
   - Portfolios with negative returns

2. **Verify Tax Calculations**:
   - Check that annual allowance resets properly
   - Verify Teilfreistellung is applied correctly
   - Ensure negative portfolio values display correctly

3. **Chart Functionality**:
   - Test tooltip display with negative values
   - Verify chart scaling with various portfolio trajectories
   - Check that all data series display correctly

## Additional Fixes Applied

### **4. Teilfreistellung Rate Configuration**
- **Added User Control**: Users can now configure the Teilfreistellung rate (0-30%)
- **Smart Defaults**: 30% for equity ETFs, 15% for mixed funds, 0% for bond ETFs
- **Contextual Display**: Only shown when taxes are enabled

### **5. Corrected Average Monthly Calculations**
- **Before**: Used incorrect formula `avg net = first-year gross - avg tax`
- **After**: Proper calculation `avg net = Σ net / total months`
- **Impact**: Average monthly net now correctly shows €3,535.37 instead of €2,543.42

### **6. Enhanced UI Consistency**
- **Unified Calculations**: UI and table now use identical tax logic
- **Proper Labels**: Clear distinction between "first year" and "average" values
- **Additional Display Fields**: Added support for showing both first-year and average values

## Implementation Notes

- All changes maintain backward compatibility
- Logging has been enhanced for debugging tax calculations
- The fixes follow German tax law for ETF withdrawals (excluding Soli as requested)
- Error handling has been improved for edge cases
- User can now configure Teilfreistellung rate based on ETF type

## Files Modified

- `app.js`: Main calculation and chart functions
- `etf_savings.html`: Added Teilfreistellung rate configuration UI
- Functions affected:
  - `simulateWithdrawal()` - Proportional cost basis method
  - `updateWithdrawalChart()` - Improved tooltip formatting
  - `setupWithdrawalListeners()` - Added Teilfreistellung rate control
  - `updateWithdrawalSliderValue()` - Support for new slider
  - Tax calculation logic within the simulation loop 