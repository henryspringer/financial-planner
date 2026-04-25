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
- If your bank's layout differs from the default expected columns, edits the checking parser (`parseCSV()`) in `index.html` to match
- May rename the function to something generic like `parseCheckingCSV()`
- Has you upload the CSV via the app's Monthly Uploads hub; data persists to localStorage

---

## 2. Credit cards — CSV per card

**Why:** categorized discretionary spending (groceries, dining, shopping, subscriptions, travel). If you use Apple Card, cardholder breakdown (e.g. spouse 1 vs. spouse 2) comes along for free.

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

> Taxable brokerage: $X (holdings mix, e.g. mostly index ETFs + a few individual stocks)
> Roth IRA (spouse 1): $Y
> Traditional IRA (spouse 2): $Z
> HSA: $W

The agent updates the input defaults in `index.html` to the values you provide. You don't need holdings-level detail unless you want the allocation view.

---

## 4. Retirement — 401(k) balances

**Why:** largest chunk of most households' net worth projection.

**What to send:**

- Current balance per spouse, per plan
- Annual contribution as a % of gross (or as a dollar amount)
- Employer match terms if relevant (e.g. "50% match up to 6%")

**Example:**

> Spouse 1: $[balance] at [provider] 401(k), contributing [X]% of $[salary]. [match terms].
> Spouse 2: $[balance] at [provider] 401(k), contributing [X]% of $[salary]. [match terms].

**What the agent updates:**

- `i-henry401k`, `i-hadley401k` input defaults (internal IDs — visible labels are "Spouse 1 / Spouse 2")
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

> Child 1: $[balance], $[X]/mo ongoing, [plan provider].
> Child 2: $[balance], $[X]/mo, [plan provider].
> Child 3: $[balance], $[X]/mo, [plan provider].

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

> Home value: ~$[X] (Zillow / Redfin / appraisal)
> Mortgage balance: $[X] at [rate]%, [N] years remaining
> PITI: $[X]/mo

**What the agent updates:**

- `i-home0` input default (computed home equity = value − mortgage)
- Housing line in `BUDGETS` to match your PITI

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
- Expected real return assumption (typical: 5–7%)
- Expected expense growth (typical: 2–3%)
- Effective tax rate estimate (typical: 25–40% depending on bracket + state)

All assumption inputs ship at 0 — set them yourself or have the agent set sensible values you confirm.

**Example:**

> Spouse 1 [age], spouse 2 [age], target retire [age].
> Combined gross $[X].
> Use [X]% real return, [X]% expense growth, [X]% tax rate.

---

## 9. Kids — childcare schedule

**Why:** the planner models childcare as a dynamic expense that drops as each kid hits free pre-K. This can be a meaningful per-child cliff that materially changes the retire-at-X math.

**What to send:**

- Each kid's name + birthdate (or current age)
- Current weekly/monthly daycare or nanny rate per child (if any)
- When they start free school (your local public pre-K age, or private school tuition if paying)
- Any known step-functions (e.g. "daycare drops to after-school care when [child] starts K in [year]")

**What the agent updates:**

- `CHILDCARE_SCHEDULE` in `index.html`
- Kid names in quarterly review copy

---

## 10. Monthly budget

The app ships with all 11 budget categories at $0. You define them. Send the agent a target per category, or a single monthly total and let the agent suggest a split:

> Set housing $X, childcare $X, groceries $X, dining $X, transport $X, travel $X, shopping $X, subs $X, health $X, golf $X, other $X.

Or:

> Total target $[X]/mo across the 11 categories — propose a split and I'll adjust.

**What the agent updates:**

- `BUDGETS` constant in `index.html`
- Total auto-recalculates from the values you provide

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
