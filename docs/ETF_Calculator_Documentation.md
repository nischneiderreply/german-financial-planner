
# ETF-Sparplan & Entnahme Rechner - Comprehensive Documentation

## Table of Contents
1. [Overview](#overview)
2. [Application Structure](#application-structure)
3. [German Tax Calculator](#german-tax-calculator)
4. [Phase 1: Budget Planning](#phase-1-budget-planning)
5. [Phase 2: Accumulation Phase & Scenario Management](#phase-2-accumulation-phase--scenario-management)
6. [Phase 3: Withdrawal Phase](#phase-3-withdrawal-phase)
7. [Mathematical Formulas](#mathematical-formulas)
8. [German Localization](#german-localization)
9. [Technical Implementation](#technical-implementation)
10. [Data Flow](#data-flow)
11. [Code Structure](#code-structure)

## Overview

The ETF-Sparplan & Entnahme Rechner is a comprehensive German financial planning application that guides users through distinct phases of financial planning:

1.  **German Tax Calculator**: Calculate net salary from gross income with full German tax compliance for 2025.
2.  **Budget Planning**: Analyze income and expenses to determine available savings, with profile management.
3.  **Accumulation Phase**: Model ETF investment growth over time, considering German tax implications (Vorabpauschale, Teilfreistellung), progressive salary/tax modeling, and multi-scenario comparison.
4.  **Withdrawal Phase**: Plan retirement withdrawals with realistic depletion modeling using precise mathematical formulas (including Abgeltungssteuer and Teilfreistellung considerations), and an integrated timeline view.

### Key Features
-   **Complete Financial Planning Suite**: From salary calculation to retirement planning.
-   **Accurate German Tax Calculator**: Based on official 2025 tax rates, including income tax, church tax, and all social insurances (pension, unemployment, health, care).
-   **Progressive Tax Modeling**: Realistic salary increase calculations in the accumulation phase, considering how higher income impacts net gains due to progressive tax brackets.
-   **Full German Localization**: Proper number formatting (comma as decimal separator) and German UI.
-   **German Tax Law Compliance**: Considers Abgeltungssteuer (capital gains tax), Sparerpauschbetrag (tax-free allowance for capital gains), Vorabpauschale (preliminary lump-sum tax for accumulating ETFs), and Teilfreistellung (partial tax exemption for equity funds) in relevant phases.
-   **Flexible Period Inputs**: Monthly/yearly input options for income and expense categories in budget planning.
-   **Interactive Charts**:
    -   Budget distribution (pie chart).
    -   Multi-scenario wealth development comparison (line chart).
    -   Accumulation phase: Einzahlungen vs. Gewinne (stacked area chart for active scenario).
    -   Withdrawal phase portfolio and cashflow analysis (line chart).
    -   Integrated timeline combining accumulation and withdrawal (line chart).
-   **Profile Management (Budget)**: Save, load, and manage multiple budget profiles using LocalStorage.
-   **Scenario Management (Accumulation)**: Create, compare, copy, and delete up to 4 investment scenarios.
-   **Seamless Integration**: Values can be transferred between phases (e.g., net salary from tax calculator to budget, savings from budget to accumulation, final capital from accumulation to withdrawal).
-   **Responsive Design**: Adapts to desktop and mobile devices.
-   **Real-time Calculations**: Instant feedback with smooth UI updates.
-   **Mathematical Precision in Withdrawal**: Uses direct formula-based calculations (Newton-Raphson iteration) for exact portfolio depletion.
-   **Transparent Calculations**: Console logging provides insights into calculation steps, especially for the withdrawal phase and tax calculations.
-   **Enhanced Validation & Error Handling**: Robust input handling.

## Application Structure

### File Organization
The application is contained within a single `public/etf_savings.html` file, which includes:
-   **HTML Structure**: Defines the layout, input fields, buttons, and containers for each phase.
-   **CSS Styling**: Embedded CSS for modern UI, responsiveness, and German-specific visual cues.
-   **JavaScript Logic**: Handles phase management, all calculations, chart rendering, event handling, and data persistence (LocalStorage for budget profiles).

### Phase Navigation
The application uses a clear top-level navigation system:

```html
<!-- Main Planning Phases -->
<div class="planning-section">
    <h3>📊 Finanzplanung & ETF-Investment</h3>
    <div class="phase-toggle main-phases">
        <button class="phase-button" id="budgetPhase">💼 Budgetplanung</button>
        <button class="phase-button active" id="accumulationPhase">📈 Ansparphase</button>
        <button class="phase-button" id="withdrawalPhase">🏖️ Entnahmephase</button>
    </div>
</div>

<!-- Separate Tax Calculator -->
<div class="tax-section">
    <h3>🇩🇪 Steuerrechner</h3>
    <div class="phase-toggle tax-calculator">
        <button class="phase-button" id="taxCalculatorPhase">🧮 Deutscher Steuer- & Gehaltsrechner</button>
    </div>
</div>
```

-   **Main Planning Phases**:
    -   `budgetPhase`: Budget Planning
    -   `accumulationPhase`: Accumulation Phase (default active)
    -   `withdrawalPhase`: Withdrawal Phase
-   **Separate Tax Calculator**:
    -   `taxCalculatorPhase`: German Tax & Salary Calculator

JavaScript (`setupPhaseToggle`) manages the visibility of corresponding sections (`budgetSection`, `taxCalculatorSection`, `top-section` + `accumulationChart` for accumulation, `withdrawalSection`). Only the active phase's content is displayed.

## German Tax Calculator

### Purpose
Calculates an accurate net salary from gross income based on official German tax regulations for 2025. This provides a realistic basis for budget planning and understanding salary progression.

### Input Parameters
Located within `#taxCalculatorSection`:
-   **Brutto-Jahresgehalt** (`#grossSalary`): Gross Annual Salary.
-   **Steuerklasse** (`#taxClass`): Tax Class I-VI.
-   **Bundesland** (`#federalState`): All 16 German states (affects church tax rate).
-   **Alter** (`#age`): Age (affects care insurance for childless individuals).
-   **Anzahl Kinder** (`#children`): Number of children (affects tax allowances and care insurance).
-   **Kirchensteuerpflichtig** (`#churchTaxToggle`): Toggle for church tax liability.
-   **Gesetzlich krankenversichert** (`#publicHealthInsuranceToggle`): Toggle for public/private health insurance. If public, an additional slider appears:
    -   **Zusatzbeitrag Krankenkasse** (`#healthInsuranceRate`): Slider for additional health insurance contribution rate (0.5% - 3.5%, default 2.5%).
-   **Jährliche Gehaltssteigerung** (`#annualSalaryIncrease`): Slider (0%-8%), used for informational display in this section but primarily impacts accumulation phase calculations.

### German Tax Calculation Engine (`calculateTaxes` function)
The calculation uses official 2025 German tax parameters:

#### Tax-Free Allowances (2025)
```javascript
const basicAllowance = 12096; // Grundfreibetrag
const childAllowance = children * 9600; // Kinderfreibetrag (€9,600 per child)
let taxableIncome = Math.max(0, grossSalary - basicAllowance - childAllowance);
```

#### Progressive Income Tax Brackets (Einkommensteuer - 2025)
```javascript
let incomeTax = 0;
if (taxableIncome > 0) {
    if (taxableIncome <= 17005) { // Zone 1 (approx. 14% to 24%)
        const y = taxableIncome / 10000;
        incomeTax = (922.98 * y + 1400) * y;
    } else if (taxableIncome <= 68429) { // Zone 2 (approx. 24% to 42%) Note: 68429 is the upper limit for this formula.
        const z = (taxableIncome - 17005) / 10000;
        incomeTax = (181.19 * z + 2397) * z + 1025.38;
    } else if (taxableIncome <= 277825) { // Zone 3 (42%)
        incomeTax = 0.42 * taxableIncome - 10602.13;
    } else { // Zone 4 (45%)
        incomeTax = 0.45 * taxableIncome - 18936.88;
    }
}
```

#### Church Tax (Kirchensteuer)
Rate is 8% or 9% of income tax, depending on `federalState`. E.g., 8% for Bayern, 9% for Nordrhein-Westfalen.

#### Social Insurance Contributions (Sozialversicherungsbeiträge - 2025)
-   **Contribution Ceilings (Beitragsbemessungsgrenzen)**:
    -   Pension/Unemployment: `maxPensionBase = 96600` (unified West/East)
    -   Health/Care: `maxHealthBase = 66150`
-   **Employee Shares**:
    -   **Pension Insurance (Rentenversicherung)**: 9.3% of `pensionBase`.
    -   **Unemployment Insurance (Arbeitslosenversicherung)**: 1.3% of `pensionBase`.
    -   **Health Insurance (Krankenversicherung)**: If public, 7.3% (base employee share) + (additional rate / 2) of `healthBase`. Private is estimated.
    -   **Care Insurance (Pflegeversicherung)**: Base employee share is 1.8% of `healthBase`.
        -   Childless individuals (age >= 23): +0.6% surcharge on the full care insurance rate.
        -   Families with 2 or more children under 25: Reduction of 0.25% per child from the 2nd to the 5th child (max 1.0% reduction from total rate). The employee share is adjusted accordingly.

### Results Display
-   **Brutto-Monatsgehalt** (`#grossMonthlySalary`)
-   **Netto-Monatsgehalt** (`#netMonthlySalary`)
-   **Netto-Jahresgehalt** (`#netYearlySalary`)
-   **Gesamte Abzüge** (`#totalDeductions`)
-   **Detailed Breakdown**: Individual amounts for `incomeTax`, `churchTax`, `healthInsurance`, `careInsurance`, `pensionInsurance`, `unemploymentInsurance`.

### Integration
-   **"Netto-Gehalt in ETF-Rechner übernehmen"** (`#useTaxCalculatorResults`):
    -   Transfers calculated net monthly salary to the "Netto-Gehalt" field (`#salary`) in Budget Planning.
    -   Transfers gross annual salary to "Aktuelles Brutto-Jahresgehalt" (`#baseSalary_A`) in the active Accumulation Phase scenario.
    -   Suggests a 20% savings rate based on the net salary for "Monatliche Sparrate" (`#monthlySavings_A`) in the active Accumulation Phase scenario.
    -   Switches view to the Accumulation Phase.

## Phase 1: Budget Planning

### Purpose
To analyze personal income and expenses, determine the monthly disposable income, and allocate a portion to savings. Supports saving and loading budget profiles.

### Components
Located within `#budgetSection`.

#### 1. Profile Controls (`.profile-controls`)
-   **Profil speichern** (`#saveProfile`): Opens a modal (`#saveProfileModal`) to save the current budget setup.
-   **Profil laden** (`#loadProfile`): Opens a modal (`#loadProfileModal`) to load a previously saved budget profile.
-   **Profile verwalten** (`#manageProfiles`): Opens a modal (`#profileModal`) to view, load, or delete saved profiles.
-   **Zurücksetzen** (`#resetBudget`): Resets all budget inputs to default values.

#### 2. Income, Fixed Costs, Variable Costs (`.expense-category`)
Each category (Einkommen, Fixkosten, Variable Kosten) has:
-   A header with the category title.
-   A **period toggle** (`#incomePeriodToggle`, `#fixedPeriodToggle`, `#variablePeriodToggle`) allowing inputs to be interpreted as "Monatlich" (Monthly) or "Jährlich" (Yearly).
-   Input fields for various items (e.g., `#salary`, `#rent`, `#food`).
-   A display for the sum of that category (e.g., `#incomeTotal`).

**Income Types**: Netto-Gehalt (`#salary`), Nebeneinkommen (`#sideIncome`), Sonstige Einkünfte (`#otherIncome`).
**Fixed Costs**: Miete/Hypothek (`#rent`), Nebenkosten (`#utilities`), Krankenkasse (`#health`), Versicherungen (`#insurance`), Internet/Telefon (`#internet`), GEZ (`#gez`).
**Variable Costs**: Lebensmittel (`#food`), Transport/Auto (`#transport`), Freizeit (`#leisure`), Kleidung (`#clothing`), Abonnements (`#subscriptions`), Sonstiges (`#miscellaneous`).

#### 3. Budget Summary (`.budget-summary`)
-   Displays **Total Income** (`#totalIncome`), **Total Expenses** (`#totalExpenses`), and **Remaining Budget** (`#remainingBudget`).
-   The background color of "Verfügbar" changes based on the amount (green for high, orange for low, red for negative).
-   **Budget Pie Chart** (`#budgetPieChart`): Visualizes the distribution of expenses and remaining budget.

#### 4. Savings Allocation (`.savings-allocation`)
-   **Sparrate festlegen**:
    -   Toggle between **Fester Betrag** (`#fixedAmount`) and **Prozentsatz** (`#percentage`) of remaining budget.
    -   Input for fixed amount (`#savingsAmount`) or percentage slider (`#savingsPercentage`).
-   **Monatliche ETF-Sparrate** (`#finalSavingsAmount`): Displays the calculated savings amount.
-   **"Sparrate in Ansparphase übernehmen"** (`#applySavingsRate`): Transfers the calculated savings amount to the "Monatliche Sparrate" field of the active Accumulation Phase scenario and switches to that phase.

### Period Conversion Logic (`convertToMonthly` function)
If a period toggle is set to "Jährlich", the corresponding input values are divided by 12 to get monthly figures for calculations.

### Budget Calculation (`calculateBudget` function)
-   Sums up all monthly income.
-   Sums up all monthly fixed and variable expenses.
-   Calculates `remainingBudget = totalIncome - totalExpenses`.
-   Updates all display fields and the budget pie chart.

### Profile Management (Modals: `#saveProfileModal`, `#loadProfileModal`, `#profileModal`)
-   **Saving**:
    -   User provides a profile name (`#profileName`) and optional description (`#profileDescription`).
    -   Current budget inputs (income, expenses, savings settings, period toggles) are serialized to JSON and stored in `localStorage` with a key like `budgetProfile_PROFILENAME`.
-   **Loading**:
    -   Lists available profiles from `localStorage` in `#loadProfileList`.
    -   Selecting a profile populates all budget fields with the saved data.
-   **Managing**:
    -   Lists profiles in `#profileList` with options to load or delete.
-   Data stored includes all input values, savings mode/percentage, and period toggle states.

## Phase 2: Accumulation Phase & Scenario Management

### Purpose
To model the long-term growth of ETF investments, allowing for comparison between multiple scenarios with different parameters. Incorporates German tax considerations (Vorabpauschale, Abgeltungssteuer, Teilfreistellung) and realistic salary progression with progressive tax impact.

### Components
Located within `.top-section` (controls and results) and `#accumulationChart` (chart).

#### 1. Scenario Management
-   **Scenario Tabs (`#scenarioTabs`)**:
    -   Allows switching between different scenarios (A, B, C, D).
    -   `#addScenarioBtn`: Adds a new scenario (up to 4). New scenarios inherit values from Scenario A.
-   **Scenario Panels (`#scenarioPanels`)**:
    -   Each scenario has its own panel (`.scenario-panel`) containing its specific input parameters.
    -   Panel Header: Displays scenario name. Includes buttons to **Copy** (`onclick="copyScenario('ID')"`) the current scenario's settings to a new one, or **Delete** (`onclick="removeScenario('ID')"`) the scenario (Scenario A cannot be deleted).

#### 2. Input Parameters (per scenario, e.g., for Scenario A)
-   **Monatliche Sparrate** (`#monthlySavings_A`): Monthly investment amount.
-   **Startkapital** (`#initialCapital_A`): Initial lump sum investment.
-   **Deutsche Abgeltungssteuer einbeziehen** (`#taxToggle_A`): Toggle with label "Deutsche Abgeltungssteuer einbeziehen (25% mit Vorabpauschale)". Enables annual tax calculation.
-   **ETF-Typ für Steuerberechnung** (`input[name="etfType"]`): Radio buttons for "Thesaurierend (Vorabpauschale)" (default) or "Ausschüttend".
-   **Teilfreistellung bei Aktienfonds anwenden** (`#teilfreistellungToggle_A`): Toggle for "30% steuerfrei". Active by default. Disabled if `#taxToggle_A` is off. Help text at `#teilfreistellungHelp_A`.
-   **Jährliche Rendite** (`#annualReturn_A`): Expected annual return (1-15%).
-   **Inflationsrate** (`#inflationRate_A`): Expected annual inflation (0-6%).
-   **Jährliche Gehaltssteigerung** (`#salaryGrowth_A`): Expected annual gross salary increase (0-8%).
-   **Anlagedauer** (`#duration_A`): Investment period in years (1-50).
-   **Aktuelles Brutto-Jahresgehalt** (`#baseSalary_A`): Current gross annual salary, used as the starting point for salary progression.
-   **Gehaltssteigerung für Sparrate** (`#salaryToSavings_A`): Percentage of *net* salary increase to be added to the monthly savings rate (0-100%).

#### 3. Salary Increase Analysis (per scenario, e.g., `#salaryIncreaseAnalysis_A`)
-   Displays a breakdown of how a gross salary increase translates to a net increase, considering progressive German taxation.
-   Shows: Brutto-Erhöhung (`.gross-increase`), Netto-Erhöhung (`.net-increase`), Steuer/Abgaben auf Erhöhung (`.tax-on-increase`), Netto-Rate der Erhöhung (`.net-increase-rate`).
-   This helps users understand the real impact of salary growth on disposable income and savings potential.

#### 4. Scenario Results Comparison (`.results-section` -> `#scenarioResults`)
-   Displays a summary card (`.scenario-result-card`) for each calculated scenario.
-   Shows: Endbetrag (Nominal), Endbetrag (Real), Gesamt Eingezahlt, Gesamtrendite, Gezahlte Steuern, Netto-Rendite.

#### 5. Wealth Development Chart (`#wealthChart`) and Chart Toggle
-   A chart toggle (`#scenarioComparisonBtn`, `#contributionsGainsBtn`) allows switching views:
    -   **Szenario-Vergleich** (default): Line chart comparing the wealth development of all active scenarios. Plots Nominal and Real portfolio value over time.
    -   **Einzahlungen vs. Gewinne**: Stacked area chart for the *active* scenario, showing "Eingezahltes Kapital", "Kursgewinne" (total value stacked on contributions), and "Realer Wert (inflationsbereinigt)".
-   Tooltips provide detailed information for each year.

### Core Calculation Engine (`runScenario` -> `calculateWealthDevelopment` function)
For each scenario:
1.  **Initialization**: Starts with `initialCapital` and `monthlySavings`. `currentSalary` is initialized with `baseSalary`. `usedSparerpauschbetrag` is reset for the simulation if it's year 1.
2.  **Yearly Loop (for `duration` years)**:
    a.  **Monthly Loop (12 months)**:
        i.  **Apply Monthly Return**: `capital * (annualReturn / 12)`.
        ii. **Add Savings**: `capital += currentMonthlySavings`. `totalInvested` is updated.
    b.  **Annual German ETF Tax Calculation (`calculateGermanETFTax`)**: If `#taxToggle_A` is active:
        i.  Determines ETF type (accumulating/distributing) and Teilfreistellung status.
        ii. **For accumulating ETFs**: Calculates `Vorabpauschale` (preliminary lump-sum tax) based on `BASISZINS = 0.0253` (for 2025), `startOfYearCapital`, and actual `capitalGains`. `Vorabpauschale = min(Basisertrag, Wertsteigerung)`.
        iii. **For distributing ETFs**: Estimates taxable distributions (simplified as 2% of capital gains).
        iv. Applies `Teilfreistellung` (e.g., 30% for equity ETFs) to the taxable amount from (ii) or (iii).
        v.  Applies `Sparerpauschbetrag` (€1,000 annual tax-free allowance).
        vi. Calculates tax based on `ABGELTUNGSSTEUER_RATE = 0.25` on the remaining taxable amount.
        vii.Deducts tax from `capital`. `yearlyTaxesPaid` and `cumulativeTaxesPaid` are updated.
    c.  **Annual Salary Increase & Savings Adjustment (End of Year, if not last year)**:
        i.  `previousNetSalary = calculateGermanNetSalary(currentSalary, ...)` using the full tax calculator logic.
        ii. `newGrossSalary = currentSalary * (1 + salaryGrowth)`.
        iii. `newNetSalary = calculateGermanNetSalary(newGrossSalary, ...)`.
        iv. `netSalaryIncrease = newNetSalary - previousNetSalary`.
        v.  `additionalSavings = (netSalaryIncrease / 12) * salaryToSavings`.
        vi. `currentMonthlySavings += additionalSavings`.
    d.  **Calculate Real Capital**: `realCapital = capital / (1 + inflationRate)^year`.
    e.  Store yearly data: `year`, `capital`, `realCapital`, `totalInvested`, `currentMonthlySavings`, `currentSalary`, `netSalary`, `taxesPaid`, `cumulativeTaxesPaid`.

The `calculateGermanNetSalary` function used here is the same detailed one from the main Tax Calculator.

## Phase 3: Withdrawal Phase

### Purpose
To plan the de-accumulation of the invested capital during retirement, calculating sustainable withdrawal amounts while considering German taxes. Uses precise mathematical formulas for accurate portfolio depletion.

### Components
Located within `#withdrawalSection`.

#### 1. Input Parameters (`.controls-section`)
-   **Verfügbares Kapital bei Renteneintritt** (`#retirementCapital`): The portfolio value at the start of retirement.
    -   Can be manually entered or synced from an Accumulation Phase scenario result using the `#scenarioSelector` and `#useAccumulationResult` button.
    -   An auto-sync mechanism (`autoSyncWithdrawalCapital` via `#manualSyncBtn` or on changes) keeps this value updated with the active accumulation scenario, shown by `#syncIndicator` and `#syncScenarioName`.
-   **Entnahmedauer** (`#withdrawalDuration`): Desired length of withdrawal period (10-60 years).
-   **Jährliche Rendite im Ruhestand** (`#postRetirementReturn`): Expected annual return during withdrawal (1-15%).
-   **Inflationsrate** (`#withdrawalInflation`): Expected annual inflation during withdrawal (0-5%).
-   **Abgeltungssteuer anwenden** (`#withdrawalTaxToggle`): Toggle to apply 25% tax on taxable gains during withdrawal.

#### 2. Withdrawal Results (`.results-section`)
-   **Monatliche Brutto-Entnahme** (`#monthlyWithdrawal`): Calculated gross monthly withdrawal in the first year.
-   **Monatliche Netto-Entnahme** (`#monthlyNetWithdrawal`): Average net monthly withdrawal after taxes.
-   **Gesamte Steuerlast** (`#totalTaxesPaid`): Total taxes paid over the withdrawal period.
-   **Reale Kaufkraft (heute)** (`#realPurchasingPower`): The purchasing power of the first year's monthly withdrawal in today's money.
-   **Tax Info Box**: Provides general information on German tax rules for ETF withdrawals.

#### 3. Chart Views
-   **Chart Toggle (`.chart-toggle`)**: Switches between:
    -   **Entnahme-Verlauf** (`#withdrawalChartBtn`): Shows portfolio value and cashflows over the withdrawal period in `#withdrawalChartView` (using `#withdrawalChart`).
    -   **Integrierte Zeitleiste** (`#integratedTimelineBtn`): Shows a combined view of the active accumulation scenario followed by the withdrawal phase in `#integratedTimelineView` (using `#integratedChart`).
-   **Withdrawal Chart (`#withdrawalChart`)**:
    -   Plots: Portfoliowert (Nominal), Portfoliowert (Real), Netto-Entnahme (annual), Jährliche Steuern.
    -   Uses two Y-axes for portfolio value and annual amounts.
-   **Integrated Timeline Chart (`#integratedChart`)**:
    -   Combines data from the active accumulation scenario and the calculated withdrawal plan.
    -   Plots: Ansparphase (Nominal & Real), Entnahmephase (Nominal & Real).
    -   Highlights the transition point from accumulation to withdrawal.
    -   Displays total years for accumulation (`#accumulationYears`) and withdrawal (`#withdrawalYears`), and transition capital (`#transitionCapital`).

#### 4. Withdrawal Table (`.withdrawal-table` -> `#withdrawalTable`)
-   Provides a year-by-year breakdown of the withdrawal plan:
    -   Jahr, Kapital Beginn, Jährliche Entnahme, Steuern, Netto-Entnahme, Kapital Ende, Realer Wert (des Kapitals am Ende des Jahres).

### Direct Mathematical Calculation Engine (`calculateWithdrawal` -> `calculateWithdrawalPlan` function)
This engine ensures the portfolio depletes to approximately €0 over the specified duration.

1.  **Determine Cost Basis**: `userDefinedTotalContributions` is calculated from the accumulation phase (sum of all monthly contributions, passed as `validatedContributions`). This is used to determine the gain portion of withdrawals.
2.  **Step 1: Calculate Base Annual Withdrawal (`calculateDirectAnnuityPayment`)**:
    -   Uses the standard annuity formula: `PMT = PV * [r(1+r)^n] / [(1+r)^n - 1]`
    -   `PV` = `initialCapital`, `r` = `annualReturn`, `n` = `duration`.
    -   This gives the constant annual withdrawal amount that would deplete the portfolio if there were no inflation or taxes.
3.  **Step 2: Adjust for Taxes and/or Inflation (Iterative Method)**:
    -   If `includeTax` is true, `adjustForTaxes` is used.
    -   If `includeTax` is false but inflation exists, `adjustForInflationAndReturns` is used.
    -   Both functions use the Newton-Raphson iterative method to find the `baseAnnualWithdrawal` that, when simulated, results in a final capital of €0. This is necessary because taxes depend on the gain portion (which changes) and inflation-adjusted withdrawals also affect the depletion rate.
4.  **Step 3: Simulate Withdrawal (`simulateWithdrawal` function)**:
    -   Performs a year-by-year simulation using the (potentially adjusted) `baseAnnualWithdrawal`.
    -   For each year:
        a.  Calculate capital after returns: `startCapital * (1 + annualReturn)`.
        b.  Calculate inflation-adjusted gross withdrawal for the current year: `baseAnnualWithdrawal * (1 + inflationRate)^(year-1)`.
        c.  **German Tax Calculation on Withdrawal (if `includeTax` is true)**:
            i.  The gain portion of the `grossAnnualWithdrawal` is determined using a simplified average cost method. The cost basis is derived from `initialCapital` (retirement capital) and an assumed `averageCostBasisPerEuro = 1.0`. Effectively, the gain proportion of any withdrawal is `(current_capital_value - initial_cost_basis_of_that_capital) / current_capital_value`.
            ii. **Teilfreistellung**: If active (currently based on Accumulation Scenario A's `#teilfreistellungToggle_A` setting), 30% of the `realizedGain` is made tax-free.
            iii. **Sparerpauschbetrag**: €1,000 annual tax-free allowance for capital gains is applied (tracked via `cumulativeTaxFreeAllowanceUsed`, reset yearly).
            iv. **Abgeltungssteuer**: 25% tax rate (`EFFECTIVE_TAX_RATE`) is applied to the remaining taxable gain. `taxesPaid` are calculated.
        d.  Calculate net withdrawal: `grossAnnualWithdrawal - taxesPaid`.
        e.  Update portfolio capital: `capitalAfterReturns - grossAnnualWithdrawal`.
        f.  Store all yearly data.
    -   Returns detailed yearly data, total taxes, average withdrawals, final capital, etc.

### German Tax Considerations in Withdrawal (as implemented in `simulateWithdrawal`)
-   **Sparerpauschbetrag**: €1,000 annual tax-free allowance for capital gains is factored in and reset each year of withdrawal.
-   **Abgeltungssteuer**: 25% tax rate applied to taxable gains after allowances and exemptions.
-   **Cost Basis / Gain Calculation**: Uses a simplified average cost method. The proportion of gain in any withdrawal is based on the overall gain in the portfolio relative to the initial `retirementCapital`. It does not implement a FIFO method.
-   **Teilfreistellung**: A 30% partial exemption for equity funds can be applied to gains. The toggle for this (`#teilfreistellungToggle_A`) is currently read from Accumulation Scenario A's settings, which might be an area for future refinement to be a withdrawal-specific setting.

## Mathematical Formulas

### Accumulation Phase
-   **Monthly Return**: `capital * (annualReturn / 12)`
-   **Annual German ETF Tax (`calculateGermanETFTax`)**:
    -   Considers ETF type (accumulating/distributing).
    -   For accumulating: Vorabpauschale = `min(startCapital * BASISZINS_2025 * 0.7, capitalGains_year)`.
    -   For distributing: Simplified, e.g., `capitalGains_year * 0.02` (as an example of distribution yield).
    -   Applies Teilfreistellung (e.g., 30% reduction of taxable amount).
    -   Applies Sparerpauschbetrag (€1,000 annually).
    -   Tax = `remaining_taxable_amount * 0.25`.
-   **Salary Progression (Net Impact)**:
    `netSalaryIncrease = calculateGermanNetSalary(newGrossSalary) - calculateGermanNetSalary(oldGrossSalary)`
    `additionalSavings = (netSalaryIncrease / 12) * salaryToSavingsPercentage`
-   **Real Value**: `nominalValue / (1 + inflationRate)^years`

### Withdrawal Phase (Direct Calculation & Simulation)
-   **Annuity Payment (PMT)**: `PV * [r(1+r)^n] / [(1+r)^n - 1]`
    -   `PV`: Present Value (initial retirement capital)
    -   `r`: Annual return rate during retirement
    -   `n`: Number of withdrawal years
-   **Inflation-Adjusted Withdrawal (Year `y`)**: `PMT_year1 * (1 + inflationRate)^(y-1)`
-   **Taxable Gain (Average Cost Method)**: `realizedGain = grossWithdrawal - costBasisOfSoldShares`. `costBasisOfSoldShares` is derived from `initialCapital` at retirement and current portfolio valuation.
-   **Tax Paid**: `(realizedGain * (1 - TeilfreistellungRate) - Sparerpauschbetrag_available) * 0.25`.
-   **Newton-Raphson Method** (for tax/inflation adjustment of PMT): `x_new = x_old - f(x) / f'(x)`
    -   `f(x)` is the final portfolio value from `simulateWithdrawal` given a trial `x` (PMT).
    -   The goal is to find `x` such that `f(x) ≈ 0`.

### General
-   **Real Interest Rate**: `(1 + nominalRate) / (1 + inflationRate) - 1`

## German Localization
-   **UI Text**: All labels, buttons, and titles are in German.
-   **Number Formatting**:
    -   `formatCurrency()`: Displays numbers as currency in German locale (e.g., "€1.234,56").
    -   `parseGermanNumber()`: Parses string inputs with German number format ("," as decimal, "." as thousands separator) into floats.
    -   `formatGermanNumber()`: Formats numbers into German string representation.
    -   `setupGermanNumberInputs()`: Enhances input fields to correctly handle German number entry.
-   **Tax System**: Implements German tax laws for income (2025) and capital gains (Abgeltungssteuer, Vorabpauschale, Teilfreistellung, Sparerpauschbetrag).

## Technical Implementation

### CSS
-   Uses CSS Grid and Flexbox for responsive layouts.
-   Styled to resemble a modern, clean interface with smooth transitions and hover effects.
-   Specific styles for active states, toggles, sliders, and modals.

### JavaScript
-   **Vanilla JavaScript**: No external frameworks (except Chart.js).
-   **Modular Functions**: Code is organized into functions for specific tasks (e.g., `calculateBudget`, `runScenario`, `calculateWithdrawalPlan`, `updateComparisonChart`).
-   **Event-Driven**: Calculations and UI updates are triggered by user interactions (input changes, button clicks).
-   **Chart.js Integration**: Used for all charts (budget pie, accumulation comparison/contributions, withdrawal line, integrated timeline). Charts are configured with German formatting for tooltips and axes.
-   **State Management**:
    -   `currentPhase`: Tracks the active application phase.
    -   `scenarios` array & `activeScenario`: Manage data for multiple accumulation scenarios.
    -   `budgetData` object: Holds current budget figures and settings.
    -   `localStorage`: Used for persisting and retrieving budget profiles.
    -   `currentChartMode`: Tracks current view in accumulation chart ('comparison' or 'contributions').
-   **Debouncing**: `debouncedRecalculateAll` and `debouncedCalculateWithdrawal` are used to prevent excessive calculations during rapid input changes (e.g., slider dragging), improving performance.
-   **Notifications**: `showNotification()` function displays temporary messages for actions like saving/loading profiles or transferring data.

## Data Flow

1.  **Tax Calculator -> Budget & Accumulation**:
    `calculateTaxes()` -> User clicks "Übernehmen" (`useTaxCalculatorResults`) -> Net salary populates Budget's `#salary`; Gross salary populates Accumulation's `#baseSalary_A`; Suggested savings (20% of net) populates `#monthlySavings_A`. View changes to Accumulation.
2.  **Budget -> Accumulation**:
    `calculateBudget()` -> User clicks "Übernehmen" (`applySavingsRate`) -> `#finalSavingsAmount` populates Accumulation's `#monthlySavings_A`. View changes to Accumulation.
3.  **Accumulation -> Withdrawal**:
    `runScenario()` for active scenario -> Final capital of selected/active scenario populates Withdrawal's `#retirementCapital` via `useAccumulationResult` button or auto-sync (`autoSyncWithdrawalCapital`). The `userDefinedTotalContributions` for withdrawal is also derived from the accumulation scenario's parameters.
4.  **Profile Management (Budget)**:
    Current budget state (`budgetData`, input values) -> `confirmSaveProfile()` -> Serialized to JSON -> `localStorage.setItem()`.
    `localStorage.getItem()` -> Deserialize JSON -> Populate budget inputs and `budgetData`.

## Code Structure

### Key Global Variables
-   `chart`, `withdrawalChart`, `budgetPieChart`, `integratedChart`: Chart.js instances.
-   `currentPhase`: String, current active section.
-   `budgetData`: Object, stores budget inputs and calculations.
-   `scenarios`: Array of objects, stores data for each accumulation scenario.
-   `activeScenario`: String, ID of the currently active accumulation scenario.
-   `usedSparerpauschbetrag`: Number, tracks allowance usage within a single accumulation scenario simulation.
-   `currentChartMode`: String, for accumulation chart view.

### Main Initialization (`DOMContentLoaded`)
-   Calls setup functions for event listeners (`setupScenarioListeners`, `setupWithdrawalListeners`, `setupBudgetListeners`, `setupTaxCalculatorListeners`, `setupChartToggleListeners`, `setupPhaseToggle`, `setupGermanNumberInputs`).
-   Performs initial calculations (`recalculateAll`, `calculateBudget`, `calculateTaxes`).
-   Initializes auto-sync for withdrawal capital.

### Core Calculation Functions
-   `calculateTaxes()`: For the German Tax Calculator section (income tax & social security).
-   `calculateBudget()`: For the Budget Planning section.
-   `recalculateAll()` (iterates `runScenario()` for each scenario): For the Accumulation Phase.
    -   `runScenario()` -> `calculateWealthDevelopment()`
        -   `calculateGermanETFTax()` (for annual ETF taxes: Vorabpauschale, Teilfreistellung, Sparerpauschbetrag).
        -   `calculateGermanNetSalary()` (helper for progressive tax on salary increases).
-   `calculateWithdrawal()`: For the Withdrawal Phase.
    -   `calculateWithdrawalPlan(..., userDefinedTotalContributions)`
        -   `calculateDirectAnnuityPayment()`
        -   `adjustForTaxes()` / `adjustForInflationAndReturns()` (using Newton-Raphson)
        -   `simulateWithdrawal(..., baseAnnualWithdrawal)` (calculates taxes on withdrawal using average cost, Teilfreistellung, Sparerpauschbetrag).

### UI Update Functions
-   `updateMainChart()` (delegates to `updateComparisonChart` or `updateContributionsGainsChart`).
-   `updateComparisonChart()` (Accumulation scenarios comparison).
-   `updateContributionsGainsChart()` (Accumulation active scenario: contributions vs. gains).
-   `updateWithdrawalChart()`
-   `updateWithdrawalTable()`
-   `updateBudgetPieChart()`
-   `createIntegratedTimeline()`
-   `updateScenarioResults()` (Accumulation scenario summary cards).
-   `updateScenarioSalaryAnalysis()`
-   Functions to update slider value displays (e.g., `updateScenarioSliderValue`, `updateWithdrawalSliderValue`).

### Event Handling
-   Listeners are attached to input fields, sliders, toggles, and buttons.
-   Input events often trigger a corresponding calculation function (sometimes debounced).
-   Button clicks trigger phase changes, data transfers, modal operations, or scenario management.

### Profile & Scenario Management Functions
-   **Budget Profiles**: `openSaveProfileModal`, `confirmSaveProfile`, `openLoadProfileModal`, `loadSelectedProfile`, `openProfileManager`, `deleteProfile`, `resetBudget`.
-   **Accumulation Scenarios**: `addNewScenario`, `createScenarioPanel`, `createScenarioTab`, `switchToScenario`, `copyScenario`, `removeScenario`.

### Utility Functions
-   `formatCurrency()`, `parseGermanNumber()`, `formatGermanNumber()`, `setupGermanNumberInputs`
-   `convertToMonthly()`
-   `showNotification()`
-   `escapeHtml()`
