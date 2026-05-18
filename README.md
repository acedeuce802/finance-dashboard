# Finance Dashboard

A single-file personal finance tracker built in vanilla HTML/JS with no dependencies beyond Chart.js. Tracks net worth over time, projects retirement, and models both Coast FIRE and Traditional FIRE scenarios.

---

## Getting Started

No installation, no server, no account required.

1. Open `finance-dashboard.html` in any modern browser
2. Add your accounts under the **Accounts** tab
3. Record your first balance snapshot on the **Dashboard** tab
4. Configure retirement settings on the **Coast FIRE** or **FIRE** tab

Data saves automatically to your browser's `localStorage` on every change. Use **Save JSON** in the nav to export a portable backup and **Load JSON** to restore it in any browser.

---

## Tabs

### Dashboard
- Net worth history chart with **1Y / 2Y / 5Y / All** range selector
- Proportional time spacing — months with no entry don't compress the chart
- Assets by category donut chart with a 5-month history table ($ and % per category)
- Account breakdown list showing latest balances
- Balance history log
- **+ Add Month Snapshot** button to record balances for any month

### Projections
- Configurable net worth projection (default: years to retirement + 10)
- Monthly contributions breakdown by category, including employer match totals
- One-time events (bonuses, windfalls, planned lump-sum contributions)
- Debt payoff timeline with months-to-payoff and total interest per account
- **Scenarios** — model "what if I increase contributions?" without changing defaults

### FIRE (Traditional)
Assumes you keep contributing at your current rate until your portfolio hits the FIRE number. Finds the earliest age you could retire.

- Earliest FIRE age stat with progress toward FIRE number
- Portfolio growth chart — blue while contributing, green after FIRE number is crossed
- **Plan Snapshots** — save today's projection and revisit later to compare planned vs actual

### Coast FIRE
You've coasted when your portfolio, left alone at its expected return rate, will grow to the FIRE number by retirement — no further contributions needed.

- Coast FIRE age (the age at which you can stop contributing)
- Chart shows contributing phase (blue) and coasting phase (gold)
- **Plan Snapshots** — same comparison feature as the FIRE tab

### Accounts
- Add/edit/delete accounts across all categories
- Drag to reorder — order carries through to all other views
- Per-account return rate, contribution amount (fixed $ or % of salary), employer match
- **Include in retirement calculations** checkbox — exclude earmarked accounts (e.g. house down payment, emergency fund) from FIRE math without deleting them

### Settings
- Name, currency symbol, annual gross salary, birth year
- Data management: export JSON, import JSON, clear all data

---

## Key Concepts

### FIRE Number
```
FIRE Number = (Monthly Spend × 12) / Safe Withdrawal Rate
```
The portfolio size needed to fund retirement indefinitely. A common starting point is a 4% SWR, meaning you need 25× your annual expenses.

### Tax Adjustment
Monthly spend in retirement is post-tax money, but most portfolios are a mix of pre-tax and post-tax accounts. Enter an **Effective Tax Rate in Retirement** and the FIRE number is grossed up so withdrawals actually cover your spending after taxes.

```
Gross Annual Withdrawal = (Monthly Spend × 12) / (1 - Tax Rate)
FIRE Number             = Gross Annual Withdrawal / SWR
```

Rough guide based on account mix:
| Portfolio composition | Suggested rate |
|---|---|
| Mostly Roth IRA / Roth 401k | 0–5% |
| Mix of Roth and traditional | 10–20% |
| Mostly traditional 401k / IRA | 20–30% |
| Taxable brokerage | 10–15% (long-term cap gains) |

### Coast FIRE vs Traditional FIRE
| | Coast FIRE | Traditional FIRE |
|---|---|---|
| Question answered | When can I *stop* contributing? | When can I *retire*? |
| Assumption after target age | No further contributions | N/A — already retired |
| Portfolio target | Coast number (shrinks as you approach retirement) | Full FIRE number |
| Key input | Retirement age | Monthly spend + SWR |

### Employer Match (Two-Tier)
Supports the common structure of an unconditional base match plus a capped dollar-for-dollar match:
```
Total Match = (Salary × Base Match %)
            + min(Employee Contrib, Salary × Match Limit %) × Match Rate %
```
Example — "4% regardless, then dollar-for-dollar up to 6%":
- Base Match: 4%
- Additional Match Rate: 100%
- Additional Match Limit: 6%

---

## Account Types

| Category | Sub-types |
|---|---|
| Bank / Savings | Checking, Savings / HYSA |
| Investment | Brokerage |
| Retirement | 401(k), 403(b), Roth IRA, Traditional IRA, HSA, Pension |
| Asset | Home, Vehicle, Other Asset |
| Debt | Mortgage, Auto Loan, Student Loan, Credit Card, Other Debt |

**Debt accounts** track balances for net worth purposes only. The app does not simulate paying down debt month-by-month in the balance log — enter updated balances manually each month. It does model amortization in the Projections tab to show payoff date and total interest.

**Credit cards** are assumed paid in full each statement period.

---

## Projection Engine

The projection simulates month-by-month from today:

- Each asset account grows at its own annual return rate (compounded monthly)
- Debt accounts follow a full amortization schedule from the latest recorded balance
- Employer match is added each month alongside employee contributions
- One-time events (bonuses, windfalls) are applied in their scheduled month
- Contributions stored as a percentage of salary re-compute automatically if salary changes

---

## Plan Snapshots

Both the FIRE and Coast FIRE tabs support saving a snapshot of your current plan. When you revisit later the app compares:

| Column | Meaning |
|---|---|
| Portfolio at Snapshot | Your portfolio value when the snapshot was saved |
| Plan Says Today | Where the snapshot projected you'd be by now |
| Actual Today | Your real current portfolio value |
| Delta vs Plan | Difference between actual and planned |

If you're behind the planned trajectory but still projected to hit the same FIRE / coast age, the snapshot says so explicitly — a portfolio shortfall today doesn't necessarily mean you'll miss your target, because compound growth over the remaining years can close the gap.

When there's a meaningful gap that changes your projected age, two options are shown:
- **Option A — Catch Up**: extra monthly contribution needed to still hit the original age
- **Option B — Current Pace**: your new projected FIRE / coast age at current contributions

---

## Data & Persistence

All data is stored in a single JSON file containing `meta`, `accounts`, `balances`, `oneTimeEvents`, `scenarios`, `coastSnapshots`, and `fireSnapshots`.

The file is **forward-compatible**: new fields added in future app versions default gracefully when loading older JSON files, and unrecognized fields in old files are ignored. You will never lose data by updating the HTML.

Recommended workflow: save a JSON backup after each monthly snapshot entry, and keep it wherever you store important documents (NAS, encrypted cloud folder, local backup drive).
