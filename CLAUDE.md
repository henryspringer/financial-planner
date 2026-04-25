# Agent instructions — Financial Planner

This is a single-file HTML financial planner. All user data lives in browser localStorage; there is no backend. Your job is to help the user personalize `index.html` for their accounts and data sources.

Read this file fully before making edits. The order of operations matters in several places.

---

## Ground rules

1. **Never hardcode the user's personal balances or account numbers into `index.html` if the repo is published publicly.** The user should input balances via the app's input fields (which persist to localStorage), not by committing them to code. Only update `value=""` defaults on inputs if the user explicitly asks you to bake their numbers in, and warn them that those will show in any public fork.
2. **Never commit CSVs.** Bank statements, credit card exports, Schwab snapshots, and anything matching `*.private.*` should stay out of git. The default `.gitignore` should cover `*.csv`, `*.private.html`, and `docs/user-data/`.
3. **All user data stays in the browser.** The only time data leaves the user's machine is when they paste into an AI chat — which they control.
4. **Ask before renaming or deleting functions.** Other parts of the dashboard may depend on them. Use Grep first.

---

## What the user will send you

The user is working from `DATA.md`. Expect some combination of:

- **Bank statement CSV** (checking)
- **Credit card CSV(s)** (Apple Card most common, may also be Chase/Amex/etc.)
- **Brokerage snapshot** (Schwab CSV or free-text balances)
- **Free-text balances** for 401(k), IRAs, 529s, home, crypto
- **Free-text personal info** (ages, income, retirement target)
- **Kids' names + ages** for the childcare schedule

They may send everything at once via the template at the bottom of `DATA.md`, or drip it in piece by piece. Either is fine.

---

## Architecture cheatsheet

Single file, three tabs, ~3,300 lines:

- **Plan tab** (top of HTML) — inputs, long-term projection, net worth chart, childcare schedule, quarterly review panel
- **Spending tab** — Apple Card + checking analysis, actuals vs budget, drill-in, outliers, quarterly spending review panel
- **Portfolio tab** — Schwab positions tracker

Key globals:

- `parsedRows` — array of checking transactions, each `{year, month, day, desc, amt, cat, type}`
- `acRows` — array of Apple Card transactions, each `{stmtYr, stmtMo, dy, merchant, amt, cat, who}`
- `BUDGETS` — 11-category monthly budget object, sums to `BUDGET_TOTAL`
- `CAT_META` — display labels + colors for the 11 categories
- `CAT_KEYS` — ordered list of category keys (the render order)

Category precedence — when modifying rules, remember:

1. `PASSTHROUGH_WITHDRAWALS` (explicit skips — inheritance pass-throughs, etc.) — first
2. Income rules (deposits from known payroll)
3. Investment flows (Coinbase, Merrill, Robinhood → skip)
4. Apple Card payment (`APPLECARD GSBANK` → skip; already captured in AC rows)
5. Large check heuristics (≥$4K → housing; $300–$1,200 → childcare)
6. Merchant string matching (order within `categorize()` matters — first match wins)
7. Fallback to `other`

For Apple Card, same story in `AC_MERCHANT_OVERRIDES` — subscriptions match before shopping (so Amazon Prime doesn't get mis-bucketed as Amazon retail).

---

## Common tasks

### Task: personalize balances and inputs

When the user sends their numbers, update the `value=""` attribute on these HTML inputs (search for the id):

| Input ID | What it is |
|---|---|
| `i-henryAge`, `i-retireAge` | ages |
| `i-henryInc`, `i-hadleyInc` | gross incomes |
| `i-henry401k`, `i-hadley401k` | 401(k) balances |
| `i-k401` | 401(k) contribution % |
| `i-k529`, `i-k529h`, `i-k529e`, `i-k529r` | 529 contribution + balances |
| `i-home0` | current home value |
| `i-btc-qty`, `i-eth-qty` | crypto quantities |
| `i-incGrow`, `i-expGrow`, `i-realRet`, `i-taxRate`, `i-homeRet` | assumptions |
| `i-retSpend` | annual spend in retirement |

**Also update:**

- Kids' names in `CHILDCARE_SCHEDULE`
- Any spouse-name references in chart legends, quarterly review copy, etc. (search for `Henry` and `Hadley`)
- Currency / locale if the user isn't in USD (harder — flag as out of scope unless asked)

**Warn the user** if they're publishing a fork: "I've baked your balances into `index.html`. Anyone who sees your fork will see these numbers. Consider keeping a private copy and shipping only placeholder values in the public repo."

### Task: adapt the checking CSV parser

`parseBangorCSV()` is hardcoded to Bangor Savings' column layout:

```
Date, Description, Withdrawal, Deposit, Type, Check #, Balance
```

If the user's bank has a different layout:

1. Read the first 5–10 rows of their CSV (ask them to paste it if needed)
2. Identify which columns hold date, description, withdrawal, deposit, type, check number
3. Edit the parser to match
4. Consider renaming to `parseCheckingCSV()` for generality
5. Also update the label in the upload hub if it says "Bangor"

Common column variations:

- **Chase:** `Transaction Date, Post Date, Description, Category, Type, Amount, Memo` (single Amount column, negative = debit)
- **Ally:** `Date, Time, Amount, Type, Description`
- **Wells Fargo:** no header row, just `Date, Amount, *, *, Description`

If the user's bank has a single signed `Amount` column instead of separate Withdrawal/Deposit, convert it inside the parser: `withdrawal = amt < 0 ? -amt : 0; deposit = amt > 0 ? amt : 0`.

### Task: adapt credit card CSV

Apple Card is the default. If the user has a different card:

1. If format has `Transaction Date, Merchant, Category, Amount`, it's close enough to Apple's layout — edit `parseAppleCardCSV()` to match column indices
2. Most issuers don't provide Henry/Hadley-style cardholder splits — drop that column gracefully (set `who: 'You'`)
3. Category names vary wildly. Update `AC_TO_BUDGET` to map the issuer's category strings to the 11 budget buckets:

```js
const AC_TO_BUDGET = {
  'Grocery': 'groceries',      // Apple Card
  'Groceries': 'groceries',    // Chase
  'Supermarkets': 'groceries', // Amex
  // ...
};
```

4. If the issuer doesn't provide categories at all, the merchant overrides do the work. Add user-specific rules to `AC_MERCHANT_OVERRIDES` as they surface.

### Task: update categorization rules

The user will say things like "Amazon should be shopping, not subs" or "add my propane company to housing."

- **For Apple Card merchants:** add to `AC_MERCHANT_OVERRIDES` — regex, subs patterns first, then shopping
- **For checking descriptions:** add to `categorize()` — search the function for the right category block and insert an `if (d.includes('VENDOR NAME')) return {cat:'X', amt: w};`
- **Remember the precedence** — subscriptions must match before general shopping, otherwise Prime Video hits Amazon's shopping rule

After adding rules, ask the user to reload the page so the categorization re-runs, and optionally drill into the affected category to verify the fix.

### Task: adjust the budget

User says "I don't golf, delete that line" or "bump travel to $600":

1. Edit `BUDGETS` in `index.html` (search for `const BUDGETS = {`)
2. If deleting a category: also remove from `CAT_META`, `CAT_KEYS`, and check `categorize()` + `AC_MERCHANT_OVERRIDES` for rules that route to it (redirect to a sensible alternate like `other`)
3. If renaming a category: coordinate across `BUDGETS`, `CAT_META`, `CAT_KEYS`, `CHK_CAT_META`, and any references in `categorize()` / `AC_TO_BUDGET` / `AC_MERCHANT_OVERRIDES`
4. `BUDGET_TOTAL` auto-computes — no manual update needed

### Task: generate a quarterly review

The user will click "Generate Quarterly Review" on the Plan or Spending tab. This opens a modal with:

- A pre-filled prompt containing their live data snapshot
- A textarea for them to paste the AI response
- A save button that stores the review in localStorage and renders it in the panel

Your role: when the user pastes the generated prompt into your chat, produce a review that matches the HTML template included in the prompt. **Do not invent numbers.** Only use what's provided in the prompt's data block. See `prompts/spending-review.md` and `prompts/plan-review.md` for the exact schemas.

---

## Things that will break if you touch them

- **The 11-category schema.** Renaming a key requires coordinated edits across `categorize()`, `AC_TO_BUDGET`, `AC_MERCHANT_OVERRIDES`, `BUDGETS`, `CAT_META`, `CAT_KEYS`, `CHK_CAT_META`, `renderActuals()`, `renderOtherBreakdown()`, and the actuals table's `<th>` row.
- **localStorage key names.** `springer_csv_YYYY`, `springer_ac_YYYY-MM`, `springer_schwab_*` — renaming orphans existing user data. Consider migrating on read if you must rename.
- **The Apple Card CSV parser.** Apple's export format is fragile and changes occasionally. If `parseAppleCardCSV()` starts failing, check the column order in a fresh export before rewriting logic.
- **Chart.js dataset shapes.** The stacked bar charts depend on consistent `stack: 's'` keys. Don't add datasets without thinking about which stack they belong to.

---

## Style

- **Dark terminal aesthetic.** JetBrains Mono for all numbers, Inter for prose.
- **All money through `fmt$()` or `fmt$exact()`** — never raw template strings like `` `$${x}` ``.
- **Budget overruns: red** (`.neg` class). **Under-budget: default text color**, not green. The only green is for income / savings (which don't appear in the budget tracker anyway).
- **Do not add emojis** to any UI copy unless the user explicitly asks.
- **Collapsible panels** use the `toggleQR()` / `toggleSQR()` / `toggleReminders()` pattern — chevron rotates 90° on expand. Match the pattern when adding new ones.
- **Panel widths** — Plan tab uses a 2-column grid (inputs + outputs). Spending and Portfolio tabs are single-column. Don't cross-pollinate.

---

## When in doubt

Ask the user. "Should I bake this balance into the HTML file, or would you rather enter it in the input field so it saves to localStorage only?" is a fine question. Making silent decisions on data privacy is not.

For non-privacy calls (renaming a category, restructuring a panel), show your plan before executing — especially if the change spans more than two files or touches the 11-category schema.
