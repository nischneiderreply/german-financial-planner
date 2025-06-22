# Product Requirements Document (PRD): Comprehensive Scenario Comparison Feature

## 📋 **Overview**

### **Product Vision**
Create a unified, comprehensive scenario comparison page that integrates all three core components (Budgetplanung, Ansparphase, Entnahmephase) into a single, intuitive interface allowing users to easily configure, compare, and visualize different financial scenarios across the complete financial planning lifecycle.

### **Problem Statement**
Currently, users must navigate between separate pages to configure budget planning, accumulation phase, and withdrawal phase parameters. This fragmented approach makes it difficult to:
- Compare different complete financial scenarios holistically
- Understand the interconnected impact of changes across all phases
- Visualize comprehensive financial projections in a unified view
- Efficiently test multiple parameter combinations

### **Success Metrics**
- Increased user engagement (time spent on platform)
- Higher scenario creation rate (users creating 3+ scenarios)
- Improved user satisfaction scores for planning capabilities
- Reduced bounce rate from feature complexity

---

## 🎯 **Core Requirements**

### **1. Unified Page Structure**

#### **1.1 Navigation Integration**
- Add new navigation button: `🔍 Szenario-Vergleich` to existing phase toggle
- Maintain current design theme with consistent styling, colors, and animations
- Position after existing phases: `💼 Budgetplanung | 📈 Ansparphase | 🏖️ Entnahmephase | 🔍 Szenario-Vergleich`

#### **1.2 Page Layout**
```html
<div class="scenario-comparison-page">
  <div class="scenario-management-section">
    <!-- Scenario tabs and controls -->
  </div>
  <div class="parameters-configuration-section">
    <!-- Unified parameter configuration -->
  </div>
  <div class="results-visualization-section">
    <!-- Comparison charts and tables -->
  </div>
</div>
```

### **2. Scenario Management System**

#### **2.1 Scenario Creation & Organization**
- **Maximum Scenarios**: 6 scenarios (A-F) for meaningful comparison without UI clutter
- **Default Scenarios**: Start with 2 pre-configured scenarios (Conservative & Aggressive)
- **Scenario Naming**: Custom names with emoji support (e.g., "🎯 Optimistisch", "🛡️ Konservativ")
- **Color Coding**: Distinct colors per scenario using existing color palette:
  - Scenario A: `#3498db` (Blue)
  - Scenario B: `#27ae60` (Green) 
  - Scenario C: `#e74c3c` (Red)
  - Scenario D: `#f39c12` (Orange)
  - Scenario E: `#9b59b6` (Purple)
  - Scenario F: `#34495e` (Dark Gray)

#### **2.2 Scenario Controls**
```html
<div class="scenario-tabs">
  <button class="scenario-tab active" data-scenario="A">🎯 Optimistisch</button>
  <button class="scenario-tab" data-scenario="B">🛡️ Konservativ</button>
  <button class="add-scenario-btn">+ Neues Szenario</button>
</div>
```

**Functionality per scenario tab:**
- **Rename**: Double-click to edit scenario name
- **Duplicate**: Right-click menu option to copy all parameters
- **Delete**: Right-click menu option (minimum 1 scenario required)
- **Active State**: Visual indication of currently selected scenario

### **3. Comprehensive Parameter Configuration**

#### **3.1 Budget Planning Parameters**
**Income Section:**
- Netto-Gehalt (monthly/yearly toggle)
- Nebeneinkommen
- Sonstige Einkünfte

**Expense Categories:**
- **Fixkosten**: Miete, Nebenkosten, Krankenkasse, Versicherungen, Internet, GEZ
- **Variable Kosten**: Lebensmittel, Transport, Freizeit, Kleidung, Abonnements, Sonstiges

**Savings Allocation:**
- Fixed amount or percentage toggle
- Auto-calculation of available savings rate

#### **3.2 Accumulation Phase Parameters**
- **Monatliche Sparrate** (€): Range 0-5000, step 50
- **Startkapital** (€): Range 0-100000, step 1000
- **Jährliche Rendite** (%): Range 1-15, step 0.1
- **Inflationsrate** (%): Range 0-6, step 0.1
- **Anlagedauer** (Jahre): Range 1-50, step 1
- **Gehaltssteigerung** (%): Range 0-8, step 0.1
- **Gehaltssteigerung → Sparrate** (%): Range 0-100, step 5
- **Steuer-Toggles**: Deutsche Abgeltungssteuer, Teilfreistellung
- **ETF-Typ**: Thesaurierend/Ausschüttend radio buttons

#### **3.3 Withdrawal Phase Parameters**
- **Verfügbares Kapital** (€): Auto-sync from accumulation phase
- **Entnahmedauer** (Jahre): Range 10-60, step 1
- **Rendite im Ruhestand** (%): Range 1-15, step 0.1
- **Inflationsrate** (%): Range 0-5, step 0.1
- **Abgeltungssteuer**: Toggle

#### **3.4 Parameter Layout**
Organize parameters in expandable sections:
```html
<div class="parameter-sections">
  <div class="parameter-section" data-section="budget">
    <h3 class="section-header">💼 Budgetplanung</h3>
    <div class="parameter-grid">
      <!-- Budget parameters -->
    </div>
  </div>
  <div class="parameter-section" data-section="accumulation">
    <h3 class="section-header">📈 Ansparphase</h3>
    <div class="parameter-grid">
      <!-- Accumulation parameters -->
    </div>
  </div>
  <div class="parameter-section" data-section="withdrawal">
    <h3 class="section-header">🏖️ Entnahmephase</h3>
    <div class="parameter-grid">
      <!-- Withdrawal parameters -->
    </div>
  </div>
</div>
```

### **4. Results Visualization & Comparison**

#### **4.1 Summary Cards Grid**
Display key metrics for each scenario in a responsive grid:
```html
<div class="scenario-results-grid">
  <div class="scenario-result-card" data-scenario="A">
    <div class="scenario-card-header">
      <h4>🎯 Optimistisch</h4>
      <div class="scenario-color-indicator"></div>
    </div>
    <div class="key-metrics">
      <div class="metric">
        <span class="metric-label">Verfügbare Sparrate</span>
        <span class="metric-value">€650/Monat</span>
      </div>
      <div class="metric">
        <span class="metric-label">Kapital bei Renteneintritt</span>
        <span class="metric-value">€1.245.000</span>
      </div>
      <div class="metric">
        <span class="metric-label">Monatliche Rente</span>
        <span class="metric-value">€3.850</span>
      </div>
      <div class="metric">
        <span class="metric-label">Gesamtertrag</span>
        <span class="metric-value">+385%</span>
      </div>
    </div>
  </div>
</div>
```

#### **4.2 Comparison Charts**
Implement multiple chart views using Chart.js:

**Chart 1: Complete Lifecycle Overview**
- Combined chart showing accumulation + withdrawal phases
- X-axis: Years (from start of savings to end of retirement)
- Y-axis: Portfolio value (€)
- Multiple datasets (one per scenario)
- Clear transition marker at retirement

**Chart 2: Phase-Specific Comparisons**
- Toggle between "Ansparphase" and "Entnahmephase" views
- Accumulation: Contributions vs. Growth comparison
- Withdrawal: Capital depletion and income streams

**Chart 3: Key Metrics Comparison**
- Radar/spider chart showing multiple dimensions:
  - Final portfolio value
  - Total contributions
  - Average annual return
  - Risk level (volatility)
  - Tax efficiency

#### **4.3 Interactive Data Table**
```html
<table class="scenario-comparison-table">
  <thead>
    <tr>
      <th>Parameter</th>
      <th class="scenario-column">🎯 Optimistisch</th>
      <th class="scenario-column">🛡️ Konservativ</th>
      <th class="scenario-column">📊 Realistisch</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Monatliche Sparrate</td>
      <td>€650</td>
      <td>€450</td>
      <td>€550</td>
    </tr>
    <tr>
      <td>Erwartete Rendite</td>
      <td>8.5%</td>
      <td>5.5%</td>
      <td>7.0%</td>
    </tr>
    <!-- Additional comparison rows -->
  </tbody>
</table>
```

### **5. Advanced Features**

#### **5.1 Quick Configuration Presets**
Pre-configured scenario templates for common situations:
- **🎯 Optimistisch**: High return (8.5%), aggressive savings (25% of income)
- **🛡️ Konservativ**: Low return (5.5%), moderate savings (15% of income)
- **📊 Realistisch**: Balanced approach (7%), standard savings (20% of income)
- **💥 Krisenfall**: Market crash simulation with recovery
- **🏠 Immobilienkauf**: Reduced savings rate after house purchase

#### **5.2 Parameter Synchronization**
- **Budget → Accumulation**: Auto-transfer available savings rate
- **Accumulation → Withdrawal**: Auto-sync final portfolio value
- **Global Parameters**: Apply inflation rate across all scenarios simultaneously

#### **5.3 Sensitivity Analysis**
Quick parameter adjustment tools:
- **Range sliders**: Test parameter ranges (e.g., return rate 5-9%)
- **Stress testing**: Apply market crash scenarios across all scenarios
- **Monte Carlo preview**: Show optimistic/pessimistic bounds

---

## 🎨 **Design Requirements**

### **6. Visual Design Alignment**

#### **6.1 Color Scheme Consistency**
Maintain existing color palette:
- Primary Blue: `#3498db`
- Success Green: `#27ae60`
- Warning Orange: `#f39c12`
- Danger Red: `#e74c3c`
- Background: `#f8f9fa`
- Text: `#2c3e50`

#### **6.2 Component Styling**
Reuse existing component styles:
- `.input-field` for all inputs
- `.slider` and `.slider-container` for ranges
- `.toggle` for boolean options
- `.section-title` for headers
- `.result-card` for metric displays


### **7. User Experience Flow**

#### **7.1 Onboarding Workflow**

1. **Configuration**: Users modify parameters for first scenario
2. **Comparison**: Add second scenario and compare results
3. **Analysis**: Review charts

#### **7.2 Interaction Patterns**
- **Real-time Updates**: Charts update immediately when parameters change
- **Progressive Disclosure**: Advanced options collapsed by default
- **Contextual Help**: Tooltips for complex parameters

---

## 🔧 **Technical Implementation**

### **8. Data Structure**

#### **8.1 Scenario Object Model**
```javascript
const scenario = {
  id: 'A',
  name: '🎯 Optimistisch',
  color: '#3498db',
  parameters: {
    budget: {
      netSalary: 3500,
      sideIncome: 0,
      expenses: {
        fixed: 1713,
        variable: 1100
      },
      savingsRate: 650
    },
    accumulation: {
      monthlySavings: 650,
      initialCapital: 3000,
      annualReturn: 8.5,
      inflationRate: 2.0,
      duration: 25,
      salaryGrowth: 3.0,
      includeTax: true,
      etfType: 'accumulating'
    },
    withdrawal: {
      retirementCapital: 1245000,
      withdrawalDuration: 25,
      postRetirementReturn: 6.0,
      inflationRate: 2.0,
      includeTax: true
    }
  },
  results: {
    budget: { availableSavings: 650 },
    accumulation: { finalValue: 1245000, totalReturn: 385 },
    withdrawal: { monthlyIncome: 3850, realValue: 2850 }
  },
  calculatedData: {
    accumulationYearlyData: [],
    withdrawalYearlyData: [],
    combinedTimeline: []
  }
};
```

#### **8.2 State Management**
```javascript
const scenarioManager = {
  scenarios: [],
  activeScenario: 'A',
  chartInstances: {},
  
  // Core methods
  addScenario(templateId = null),
  removeScenario(scenarioId),
  duplicateScenario(scenarioId),
  updateParameter(scenarioId, path, value),
  recalculateScenario(scenarioId),
  recalculateAll(),
  
  // Comparison methods
  compareScenarios(scenarioIds),
  exportComparison(format),
  saveConfiguration(name)
};
```

### **9. Integration Points**

#### **9.1 Existing Code Reuse**
- **Budget calculations**: Reuse existing `calculateBudget()` functions
- **Accumulation math**: Leverage `calculateWealthDevelopment()` logic
- **Withdrawal math**: Utilize existing withdrawal calculation functions
- **Chart rendering**: Extend current Chart.js implementations
- **Tax calculations**: Reuse German tax logic and Abgeltungssteuer

#### **9.2 New Components Required**
- **Scenario tab management**: Create/delete/rename scenarios
- **Parameter synchronization**: Budget → Accumulation → Withdrawal
- **Multi-dataset charts**: Comparison visualizations
- **Export functionality**: PDF/Excel generation for comparisons
- **Preset management**: Save/load scenario templates

### **10. Performance Considerations**

#### **10.1 Calculation Optimization**
- **Debounced Updates**: 300ms delay for real-time parameter changes
- **Selective Recalculation**: Only recalculate changed scenarios
- **Background Processing**: Use Web Workers for heavy calculations
- **Caching**: Store calculation results until parameters change

#### **10.2 UI Performance**
- **Virtual Scrolling**: For large data tables
- **Chart Optimization**: Use Chart.js animation disable for real-time updates
- **Lazy Loading**: Load chart data only when section is visible
- **Memory Management**: Properly dispose of chart instances

---

## 📊 **Feature Validation**

### **11. User Testing Scenarios**

#### **11.1 Primary Use Cases**
1. **Young Professional**: Compare aggressive vs. conservative investment strategies
2. **Mid-Career**: Evaluate impact of salary increases on retirement outcomes
3. **Pre-Retirement**: Test different withdrawal strategies and durations
4. **Financial Advisor**: Create multiple client scenarios for presentation

#### **11.2 Edge Cases**
- **Maximum Scenarios**: Test performance with 6 active scenarios
- **Extreme Parameters**: Very high/low values for all inputs
- **Market Crash**: Negative returns and recovery scenarios
- **Inflation Spike**: High inflation impact across phases
- **Tax Changes**: Different tax scenarios and optimizations

### **12. Success Criteria**

#### **12.1 Functional Requirements**
- [ ] All existing calculations work correctly in scenario format
- [ ] Parameter changes reflect immediately in charts and summaries
- [ ] Scenarios can be created, modified, duplicated, and deleted
- [ ] Budget parameters automatically flow to accumulation phase
- [ ] Accumulation results automatically sync to withdrawal phase
- [ ] Charts display multiple scenarios with distinct colors and labels

#### **12.2 Performance Requirements**
- [ ] Page loads within 2 seconds on standard broadband
- [ ] Parameter changes update UI within 300ms
- [ ] All calculations complete within 1 second for 6 scenarios
- [ ] Memory usage remains below 100MB for extended sessions
- [ ] Charts render smoothly at 60fps during interactions

#### **12.3 Usability Requirements**
- [ ] Users can create and compare 3 scenarios within 5 minutes
- [ ] Interface remains intuitive without training for existing users
- [ ] All features accessible via keyboard navigation
- [ ] Design maintains consistency with existing pages

---

## 🚀 **Implementation Priority**

### **13. Development Phases**

#### **Phase 1: Core Infrastructure** (Week 1-2)
- Implement scenario object model and state management
- Create basic scenario tab interface
- Port existing calculations to scenario-based architecture

#### **Phase 2: Parameter Configuration** (Week 3-4)
- Build unified parameter configuration interface
- Implement parameter synchronization between phases
- Create real-time calculation updates

#### **Phase 3: Visualization** (Week 5-6)
- Develop multi-scenario charts
- Create comparison summary cards
- Implement interactive data table

#### **Phase 4: Advanced Features** (Week 7-8)
- Add preset templates and quick configuration
- Implement export/import functionality
- Optimize performance and add error handling

#### **Phase 5: Polish & Testing** (Week 9-10)
- Comprehensive testing across browsers and devices
- User experience refinements
- Documentation and help system

---

## 📝 **Conclusion**

This comprehensive scenario comparison feature will transform the ETF calculator from a single-use tool into a powerful financial planning platform. By integrating all three phases (Budgetplanung, Ansparphase, Entnahmephase) into a unified comparison interface, users will gain unprecedented insight into their complete financial lifecycle.

The implementation maintains full backward compatibility with existing functionality while providing an intuitive, visually appealing interface for complex scenario analysis. The modular architecture ensures maintainability and extensibility for future enhancements.

**Key Success Factors:**
1. **Seamless Integration**: Builds upon existing proven calculations and design patterns
2. **User-Centric Design**: Intuitive interface that reduces complexity while adding power
3. **Performance Optimization**: Real-time updates without sacrificing responsiveness
4. **Comprehensive Coverage**: Addresses complete financial planning lifecycle
5. **Future-Ready Architecture**: Extensible foundation for advanced features

This feature positions the application as a comprehensive financial planning solution suitable for both individual users and financial advisors, significantly expanding its market potential and user value proposition.