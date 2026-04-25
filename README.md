# Financial Planner

Single-file HTML financial planner. All data stays in your browser. Built to be personalized by an AI agent — you provide the statements, the agent adapts the code to your accounts.

No backend. No tracking. No accounts to create. Fork it, personalize it, run it locally forever.

## What it does

- **Plan tab** — long-term retirement projection. Net worth trajectory, SWR-based target at retirement, 401K / 529 / brokerage / home equity all modeled together. Dynamic childcare schedule that dials down as kids age into free pre-K.
- **Spending tab** — rolling 12-month actuals pulled from your checking + credit card CSVs, bucketed into 11 categories, compared against a $12,000/mo budget target. Drill-in panel lets you inspect any category for any month.
- **Portfolio tab** — taxable brokerage tracker from Schwab position snapshots, with allocation drift and concentration risk callouts.
- **Quarterly reviews** — "Generate Quarterly Review" button produces a written review of your spending or net-worth trajectory by handing your data snapshot to your AI agent of choice. The app ships with prompt templates.

## How to use (10 minutes)

### 1. Download

Grab `index.html` from this repo. Open it in any modern browser. You'll see an empty dashboard — that's expected.

### 2. Gather your data

Open [`DATA.md`](DATA.md) and collect the items listed:

- Checking account CSV (last 12 months)
- Credit card CSVs
- Brokerage / 401K / 529 balances
- Home value, mortgage balance
- Personal info (ages, income, target retirement age)

Skip anything you don't have. The dashboard degrades gracefully — missing data just means emptier panels, not a broken app.

### 3. Start an agent session

Open the repo folder in [Claude Code](https://claude.com/claude-code), [Cursor](https://cursor.sh), Continue.dev, or similar. The agent will automatically read [`CLAUDE.md`](CLAUDE.md) at the repo root and know what to edit.

No AI tooling? You can paste the contents of `CLAUDE.md` + your data into any chat-based model (Claude.ai, ChatGPT, Gemini). The agent will return edits you paste back into `index.html`.

### 4. Hand over your data

Say something like:

> Personalize this planner for me. Here are my details — [paste/attach from DATA.md]

The agent will:

- Adapt the CSV parser to your bank's column layout
- Update input defaults in `index.html` to your balances
- Adjust category rules for your vendors / subscriptions
- Guide you through uploading the CSVs via the in-app upload hub

### 5. Use it

Refresh the browser. Dashboard renders with your real numbers.

Every quarter, click **Generate Quarterly Review** on the Plan or Spending tab. The modal gives you a pre-filled prompt to paste into your AI agent — response goes back into the modal, and renders as a review panel saved to localStorage.

## Privacy

- **Zero backend.** The entire app is static HTML/JS. No network calls except for optional live crypto prices (CoinGecko).
- **Nothing leaves your browser** except what you explicitly copy/paste to your chosen AI agent.
- **Data persists in localStorage.** Clearing browser data = fresh start.
- **The repo contains zero personal info.** Safe to fork publicly.

If you publish your fork, never commit CSVs, balance screenshots, or anything matching `*.private.*`. The default `.gitignore` blocks common patterns.

## Files

```
financial-planner/
├── index.html              The entire app
├── README.md               This file
├── DATA.md                 Checklist of what to send the agent
├── CLAUDE.md               Agent instructions (read automatically by AI tools)
├── prompts/
│   ├── spending-review.md  Quarterly spending review prompt
│   └── plan-review.md      Quarterly plan review prompt
└── LICENSE                 MIT
```

## Customizing

- **Budget categories / amounts** — edit `BUDGETS`, `CAT_META`, `CAT_KEYS` in `index.html`. Must keep the three in sync.
- **Childcare schedule** — edit `CHILDCARE_SCHEDULE`. Handles the pre-K cliff when kids age into free care.
- **Merchant rules** — edit `AC_MERCHANT_OVERRIDES` (Apple Card) and `categorize()` (checking). Order matters; see `CLAUDE.md` for rules.
- **Planning assumptions** — real return, expense growth, tax rate all live in the Plan tab inputs and persist to localStorage.

Or just ask your agent to make any of these changes.

## License

MIT. Do anything you want with it.

## Not financial advice

This is a personal spreadsheet in fancier clothes. It is not licensed financial advice. Confirm tax strategy, retirement assumptions, and investment changes with a CPA or fiduciary before acting.
