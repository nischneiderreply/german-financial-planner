Looking at your ETF calculator, I can see you already have some integration between phases, but there are many opportunities to create a more seamless, connected experience. Let me search through your code to understand the current integration points and then suggest improvements.

Based on my analysis of your ETF calculator, I can see you already have some basic integration between phases, but there's tremendous potential to create a much more seamless, connected experience. Here are comprehensive ideas to enhance the integration between **Ansparphase** and **Entnahmephase**:

## 🔄 **Current Integration (What You Have)**

1. **Basic Value Transfer**: Dropdown to select scenario + button to transfer final capital
2. **Manual Process**: User must manually select scenario and click transfer button

## 🚀 **Enhanced Integration Ideas**

### 1. **Automatic Real-Time Synchronization**
```javascript
// Auto-sync withdrawal capital when accumulation results change
function autoSyncWithdrawalCapital() {
    const activeScenario = getActiveScenario();
    if (activeScenario && activeScenario.yearlyData) {
        const finalCapital = activeScenario.yearlyData[activeScenario.yearlyData.length - 1].capital;
        document.getElementById('retirementCapital').value = Math.round(finalCapital).toLocaleString('de-DE');
        
        // Show sync indicator
        showSyncIndicator('Entnahmekapital automatisch aktualisiert');
        
        // Auto-recalculate withdrawal if user is on withdrawal phase
        if (currentPhase === 'withdrawal') {
            calculateWithdrawal();
        }
    }
}
```

### 2. **Smart Parameter Inheritance**
Transfer not just capital, but intelligent parameter suggestions:

```javascript
function inheritIntelligentParameters(scenario) {
    // Inherit inflation rate (same economic environment)
    const accumulationInflation = scenario.inputs.inflationRate;
    document.getElementById('withdrawalInflation').value = accumulationInflation * 100;
    
    // Suggest lower return rate for retirement (conservative approach)
    const retirementReturn = Math.max(1, scenario.inputs.annualReturn * 0.7); // 30% lower
    document.getElementById('postRetirementReturn').value = retirementReturn * 100;
    
    // Inherit tax settings
    const includeTax = scenario.inputs.includeTax;
    document.getElementById('withdrawalTaxToggle').classList.toggle('active', includeTax);
    
    // Calculate suggested withdrawal duration based on age
    const currentAge = getCurrentAge(); // From user profile
    const suggestedDuration = Math.min(60, Math.max(15, 90 - currentAge));
    document.getElementById('withdrawalDuration').value = suggestedDuration;
}
```

### 3. **Seamless Phase Transitions with Context**
Create smooth transitions that maintain context:

```javascript
function transitionToWithdrawalPhase(scenarioId) {
    // 1. Transfer all relevant data
    inheritAllParameters(scenarioId);
    
    // 2. Show transition animation with progress
    showTransitionAnimation('Ansparphase → Entnahmephase');
    
    // 3. Switch to withdrawal phase
    document.getElementById('withdrawalPhase').click();
    
    // 4. Highlight what was transferred
    highlightTransferredValues();
    
    // 5. Show summary of transition
    showTransitionSummary(scenarioId);
}
```

### 4. **Integrated Timeline Visualization**
Show both phases on a single timeline:

```javascript
function createIntegratedTimeline() {
    // Combined chart showing:
    // - Years 0-30: Accumulation (growing portfolio)
    // - Years 30-55: Withdrawal (depleting portfolio)
    // - Transition point clearly marked
    // - Different colors/styles for each phase
    
    const combinedData = {
        labels: [...accumulationYears, ...withdrawalYears],
        datasets: [
            {
                label: 'Ansparphase',
                data: accumulationData,
                borderColor: '#27ae60',
                backgroundColor: 'rgba(39, 174, 96, 0.1)'
            },
            {
                label: 'Entnahmephase', 
                data: withdrawalData,
                borderColor: '#e74c3c',
                backgroundColor: 'rgba(231, 76, 60, 0.1)'
            }
        ]
    };
}
```

### 5. **What-If Scenario Comparisons**
Compare different accumulation scenarios and their withdrawal impacts:

```javascript
function compareScenarioImpacts() {
    // Show table comparing:
    // - Scenario A: €800k → €3,200/month for 25 years
    // - Scenario B: €1.2M → €4,800/month for 25 years  
    // - Scenario C: €600k → €2,400/month for 25 years
    
    const comparisonTable = scenarios.map(scenario => ({
        name: scenario.name,
        finalCapital: scenario.finalCapital,
        monthlyWithdrawal: calculateMonthlyWithdrawal(scenario.finalCapital),
        totalWithdrawn: calculateTotalWithdrawn(scenario.finalCapital),
        yearsOfIncome: calculateYearsOfIncome(scenario.finalCapital)
    }));
}
```

### 6. **Retirement Readiness Dashboard**
Create a comprehensive overview connecting both phases:

```javascript
function createRetirementDashboard() {
    return {
        // Accumulation Summary
        yearsToRetirement: calculateYearsToRetirement(),
        projectedCapital: getProjectedCapital(),
        monthlyContributions: getCurrentMonthlySavings(),
        
        // Withdrawal Projections
        monthlyIncome: calculateMonthlyIncome(),
        incomeReplacementRatio: calculateReplacementRatio(),
        inflationProtection: calculateInflationProtection(),
        
        // Risk Analysis
        shortfallRisk: calculateShortfallRisk(),
        longevityRisk: calculateLongevityRisk(),
        inflationRisk: calculateInflationRisk()
    };
}
```

### 7. **Smart Optimization Suggestions**
AI-like suggestions to optimize the transition:

```javascript
function generateOptimizationSuggestions(scenario) {
    const suggestions = [];
    
    // Analyze accumulation phase
    if (scenario.finalCapital < 500000) {
        suggestions.push({
            type: 'warning',
            title: 'Niedrige Rente',
            message: 'Mit €' + scenario.finalCapital.toLocaleString() + ' können Sie nur €' + 
                    calculateMonthlyWithdrawal(scenario.finalCapital) + '/Monat entnehmen.',
            action: 'Sparrate um €200/Monat erhöhen für €800/Monat mehr Rente'
        });
    }
    
    // Analyze withdrawal strategy
    if (getWithdrawalDuration() < 20) {
        suggestions.push({
            type: 'info',
            title: 'Kurze Entnahmedauer',
            message: 'Bei nur ' + getWithdrawalDuration() + ' Jahren Entnahme bleibt Geld übrig.',
            action: 'Entnahmedauer auf 30 Jahre verlängern für €' + calculateIncrease() + '/Monat mehr'
        });
    }
    
    return suggestions;
}
```

### 8. **Bidirectional Parameter Adjustment**
Allow reverse engineering - start with desired retirement income:

```javascript
function reverseEngineerSavings(desiredMonthlyIncome) {
    // Calculate required capital for desired income
    const requiredCapital = calculateRequiredCapital(desiredMonthlyIncome);
    
    // Calculate required monthly savings to reach that capital
    const requiredMonthlySavings = calculateRequiredSavings(requiredCapital);
    
    // Update accumulation phase with new targets
    updateAccumulationTargets(requiredMonthlySavings, requiredCapital);
    
    // Show the gap analysis
    showGapAnalysis(requiredMonthlySavings, getCurrentSavings());
}
```

### 9. **Integrated Stress Testing**
Test both phases under different scenarios:

```javascript
function runIntegratedStressTest() {
    const stressScenarios = [
        { name: 'Marktcrash', accumulationReturn: 0.04, withdrawalReturn: 0.03 },
        { name: 'Hohe Inflation', inflation: 0.05 },
        { name: 'Längere Lebenserwartung', withdrawalDuration: 35 },
        { name: 'Frühe Rente', accumulationDuration: 25 }
    ];
    
    stressScenarios.forEach(scenario => {
        const results = runFullLifecycleTest(scenario);
        displayStressTestResults(scenario.name, results);
    });
}
```

### 10. **Visual Connection Indicators**
Show visual connections between phases:

```css
/* Visual flow indicators */
.phase-connection {
    position: relative;
}

.phase-connection::after {
    content: '→';
    position: absolute;
    right: -20px;
    top: 50%;
    font-size: 2rem;
    color: #3498db;
    animation: pulse 2s infinite;
}

.data-flow-indicator {
    background: linear-gradient(90deg, #27ae60, #3498db);
    height: 3px;
    animation: flow 3s infinite;
}
```

### 11. **Contextual Tooltips and Guidance**
Smart help that understands the relationship:

```javascript
function showContextualGuidance() {
    if (isOnWithdrawalPhase() && hasAccumulationData()) {
        showTooltip('💡 Tipp: Ihre Ansparphase ergab €' + getFinalCapital().toLocaleString() + 
                   '. Das entspricht etwa €' + getMonthlyIncome() + '/Monat für ' + 
                   getWithdrawalDuration() + ' Jahre.');
    }
}
```

### 12. **Export Integrated Reports**
Generate comprehensive reports covering both phases:

```javascript
function generateIntegratedReport() {
    return {
        executiveSummary: generateExecutiveSummary(),
        accumulationPhase: generateAccumulationReport(),
        transitionAnalysis: generateTransitionAnalysis(),
        withdrawalPhase: generateWithdrawalReport(),
        riskAnalysis: generateRiskAnalysis(),
        recommendations: generateRecommendations(),
        charts: generateAllCharts()
    };
}
```

## 🎯 **Implementation Priority**

**High Priority (Quick Wins):**
1. Auto-sync withdrawal capital when accumulation changes
2. Smart parameter inheritance (inflation, tax settings)
3. Visual connection indicators
4. Contextual tooltips

**Medium Priority:**
5. Integrated timeline visualization
6. Retirement readiness dashboard
7. Optimization suggestions

**Advanced Features:**
8. Bidirectional parameter adjustment
9. Integrated stress testing
10. Comprehensive reporting

These enhancements would transform your calculator from separate tools into a truly integrated financial planning suite, providing users with a seamless journey from savings planning to retirement income planning.