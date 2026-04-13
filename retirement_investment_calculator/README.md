# Retirement Investment Calculator

## Features

- Projects corpus growth year-by-year through accumulation and withdrawal phases
- Answers two core questions: is your target retirement age feasible, and what is the minimum SIP you need today
- Accounts for SIP step-up, inflation-adjusted expenses, and different pre/post-retirement return assumptions
- Configurable life expectancy
- Fully client-side — your numbers never leave the browser



## Inputs

| Field | Description |
|---|---|
| Current corpus | Existing invested/saved amount (₹) |
| Monthly SIP | Current monthly investment amount (₹) |
| SIP step-up | Annual percentage increase in SIP |
| Current age | Your age today |
| Retirement age | Target age to stop working |
| Life expectancy | Age up to which expenses are funded |
| Monthly expenditure | Current monthly living expenses (₹) |
| Pre-retirement ROI | Expected annual return on equity/growth portfolio |
| Post-retirement ROI | Expected annual return after moving to safer instruments |
| Expected inflation | Annual inflation rate used to project future expenses |



## How the math works

### 1. Corpus accumulation (pre-retirement)

Monthly compounding with an annually stepped-up SIP. For each year until retirement, for each month:

```
corpus = corpus × (1 + roiPre/12) + SIP
SIP    = SIP × (1 + stepUp%)
```

### 2. Required corpus at retirement

Monthly expenditure is first inflated to the retirement date:

```
monthlyExpAtRetirement = monthlyExp × (1 + inflation)^yearsToRetirement
```

The total corpus needed to sustain this (growing with inflation) through life expectancy is the **present value of a growing annuity**:

```
PV = PMT × [1 − ((1 + g) / (1 + r))^n] / (r − g)
```

where:

- `PMT` = annualExpAtRetirement (= monthlyExpAtRetirement × 12)
- `g` = annual inflation rate
- `r` = post-retirement annual ROI
- `n` = years in retirement (life expectancy − retirement age)

Edge case: when `r ≈ g`, `PV = PMT × n`

### 3. Minimum SIP

Binary search (64 iterations) over the SIP value that makes the simulated corpus at retirement exactly equal to the required corpus. Step-up percentage is held constant.

### 4. Post-retirement simulation

Annual compounding with inflation-growing withdrawals. For each year in retirement:

```
corpus        = corpus × (1 + roiPost) − annualExpense
annualExpense = annualExpense × (1 + inflation)
```

If the corpus turns negative before life expectancy, the calculator flags the age at which funds run out.

> **Why does the corpus sometimes grow during retirement?**
> When `corpusAtRetirement × roiPost > annualExpenseAtRetirement`, annual returns exceed withdrawals and the portfolio grows. This happens when you have significantly more than the required corpus — a surplus that keeps compounding. Increase monthly expenditure or lower post-retirement ROI to see the more familiar depletion curve.



## Assumptions and limitations

- Life expectancy is the planning horizon, not a mortality prediction — use a conservatively high value
- Post-retirement simulation uses annual (not monthly) compounding, which slightly understates returns; the difference is small at typical withdrawal rates
- Inflation is applied uniformly to all expenses; large one-time costs (healthcare, weddings) are not modelled
- Returns are assumed constant — sequence-of-returns risk is not modelled
- All figures are in nominal terms (not inflation-adjusted); the required corpus formula accounts for this correctly
