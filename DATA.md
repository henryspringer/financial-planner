# What to send the agent

Gather these items once, hand them to your AI agent, and the agent will personalize `index.html` for you. Skip anything you don't have — the app degrades gracefully.

Re-run this list every quarter (or whenever your balances meaningfully change) to keep the dashboard honest.

## Summary checklist

- [ ] Checking account CSV
- [ ] Credit card CSV(s)
- [ ] Brokerage / taxable account balance
- [ ] 401(k) balances (per spouse)
- [ ] IRA balances (Roth, traditional)
- [ ] 529 balances (per child)
- [ ] Home value + mortgage balance
- [ ] Crypto holdings (if any)
- [ ] Personal info (ages, income, target retirement age)
- [ ] Kids' names + ages (for childcare schedule)

---

## 1. Checking account — CSV

**Why:** powers the entire Spending tab, the actuals vs. budget table, and the checking portion of monthly totals.

**Where to get it:**

- Log into your bank's web portal
- Navigate to the checking account → Transactions or Activity
- Export as CSV for the last 12+ months
- Most banks call this "Download transactions" or "Export activity"

**What to send the agent:**

- The CSV file itself (attach it, or paste the first ~10 rows so the agent can see the column layout)
- The name of your bank

**What the agent does:**

- Inspects the CSV header row to identify date / description / amount / type columns
- If your bank's layout differs from the default (Bangor Savings), edits `parseBangorCSV()` in `index.html` to match
- May rename the function to something generic like `parseCheckingCSV()`
- Has you upload the CSV via the app's Monthly Uploads hub; data persists to localStorage

---

## 2. Credit cards — CSV per card

**Why:** categorized discretionary spending (groceries, dining, shopping, subscriptions, travel). If you use Apple Card, cardholder breakdown (Henry vs. Hadley) comes along for free.

**Where to get it:**

- **Apple Card:** iPhone → Wallet app → card → monthly statement → Share → Export Transactions
- **Chase / Amex / Capital One / Discover:** web portal → Statements & Activity → Download CSV
- **Other:** most issuers offer CSV export somewhere in the statements section

**What to send the agent:**

- One CSV per card (or per month for Apple Card — it exports monthly)
- Which card is which (so the agent knows what cardholder / vendor rules to expect)

**What the agent does:**

- If Apple Card: uses the existing `parseAppleCardCSV()` — no parser changes needed
- If another issuer: adapts the parser, or suggests converting to a generic `(date, merchant, amount, category)` CSV
- Updates `AC_TO_BUDGET` mapping if your issuer uses different native category names

---

## 3. Brokerage / taxable accounts

**Why:** drives the Portfolio tab (allocation, concentration risk) and feeds the net-worth projection on the Plan tab.

**Two options:**

### Option A — CSV snapshot (best if you're at Schwab)

- Log into Schwab → Accounts → Positions → Export → CSV
- The app ships with a Schwab parser; upload via the Portfolio tab

### Option B — Just tell the agent the numbers

For Fidelity, Vanguard, Robinhood, or anywhere else, simply say:

> Taxable brokerage: $X (mostly VTI + individual tech stocks)
> Roth IRA (Hadley): $Y
> Traditional IRA (Henry): $Z
> HSA: $W

The agent updates the input defaults. You don't need holdings-level detail unless you want the allocation view.

---

## 4. Retirement — 401(k) balances

**Why:** largest chunk of most households' net worth projection.

**What to send:**

- Current balance per spouse, per plan
- Annual contribution as a % of gross (or as a dollar amount)
- Employer match terms if relevant (e.g. "50% match up to 6%")

**Example:**

> Henry: $180K at Atlassian 401(k), contributing 10% of $150K salary. 50% match up to 6%.
> Hadley: $124K at Shopify 401(k), contributing 8% of $135K salary. Dollar-for-dollar match up to 5%.

**What the agent updates:**

- `i-henry401k`, `i-hadley401k` input defaults
- `i-k401` (contribution %)
- `i-henryInc`, `i-hadleyInc` (gross income)

---

## 5. 529 plans — per child

**Why:** college savings leg of the projection + the "start early" reminders.

**What to send:**

- Current balance per child
- Monthly contribution per child
- Plan provider (optional — helpful if you want state tax deduction flagged)

**Example:**

> Harlow: $4,200 balance, $200/mo ongoing, Maine NextGen plan.
> Emme: $0, starting $200/mo May 1.
> Reese: $0, starting $200/mo May 1.

**What the agent updates:**

- `i-k529h`, `i-k529e`, `i-k529r` balances
- `i-k529` (annual contribution per child)
- Renames kids in `CAT_META`, childcare schedule, and quarterly review labels

---

## 6. Home

**What to send:**

- Current estimated value (Zillow / Redfin / last appraisal — close enough)
- Remaining mortgage principal balance
- Monthly PITI payment (principal + interest + taxes + insurance)
- Mortgage rate (optional — helps model payoff)

**Example:**

> Home value: ~$850K (Zillow)
> Mortgage balance: $620K at 3.125%, 26 years remaining
> PITI: $3,800/mo

**What the agent updates:**

- `i-home0` input default
- Housing line in `BUDGETS` if your PITI materially differs from $4,800/mo target

---

## 7. Crypto (skip if none)

**What to send:**

- BTC quantity
- ETH quantity
- Any other coins (the app currently tracks BTC + ETH — agent can extend for others)

Live prices are auto-fetched from CoinGecko on a button click; you don't need to provide prices.

---

## 8. Personal inputs

**What to send:**

- Your age, spouse's age
- Target retirement age
- Combined household gross income (or per-spouse)
- Expected real return assumption (default: 6.5%)
- Expected expense growth (default: 2.5%)
- Tax rate estimate (default: 35%)

**Example:**

> Henry 37, Hadley 35, target retire 55.
> Combined gross $285K.
> Happy with default assumptions.

---

## 9. Kids — childcare schedule

**Why:** the planner models childcare as a dynamic expense that drops as each kid hits free pre-K. This is often a ~$1,400/mo cliff per child that substantially changes the retire-at-X math.

**What to send:**

- Each kid's name + birthdate (or current age)
- When they start free school (your local public pre-K age, or private school tuition if paying)
- Any known step-functions (e.g. "daycare drops to after-school care when Harlow starts K in 2027")

**What the agent updates:**

- `CHILDCARE_SCHEDULE` in `index.html`
- Kid names in quarterly review copy

---

## 10. Monthly budget (optional — defaults shipped)

The app ships with a $12,000/mo budget across 11 categories, calibrated to a growing family with young kids in New England. If your target differs, say so:

> Bump travel to $600, cut golf to $0, everything else is fine.

**What the agent updates:**

- `BUDGETS` constant in `index.html`
- Total auto-recalculates

---

## Sending it all at once — template

Copy this, fill in what you have, paste into your agent chat:

```
Personalize the planner for me. Here's my data:

PEOPLE
- [Name] age [X], spouse [Name] age [X]
- Target retirement age [X]
- Household gross income: $[X]

KIDS
- [Name] (age [X]): daycare [$X/mo], starts free pre-K [YYYY]
- [Name] (age [X]): ...

RETIREMENT
- 401(k) [spouse 1]: $[X] at [provider], contributing [X%]
- 401(k) [spouse 2]: $[X] at [provider], contributing [X%]
- IRAs: [details]

INVESTMENTS
- Taxable brokerage: $[X]
- Crypto: [X BTC], [X ETH]

529s
- [Kid 1]: $[X] balance, $[X]/mo
- [Kid 2]: ...

HOME
- Value: $[X]
- Mortgage balance: $[X]
- PITI: $[X]/mo

FILES ATTACHED
- Checking CSV from [bank]
- Credit card CSV from [issuer]
- Schwab positions CSV (if applicable)

BUDGET
- [accept defaults / specify overrides]
```

That's it. Ten minutes of gathering, one agent conversation, and the planner is yours.
