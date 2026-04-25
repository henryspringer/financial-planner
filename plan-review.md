# Quarterly Plan Review — prompt template

This template is consumed by the "Generate Quarterly Review" button on the Plan tab. The app substitutes `{{placeholders}}` with live data from localStorage at click time, copies the result to the user's clipboard, and the user pastes it into their AI agent.

The agent returns HTML matching the schema below, which the user pastes back into the modal. The app saves it to localStorage under `planner_qr_plan_{{QUARTER_ID}}` and renders it in the panel.

---

## Prompt (with substitutions)

```
You are a fee-only financial planner conducting a quarterly plan review
for a household using the attached personal finance dashboard. You are
not a licensed advisor — produce analysis, not regulated advice.

Do not invent numbers. Use only the data provided below.

## The quarter
{{QUARTER_LABEL}} — generated {{GEN_DATE}}

## Household
- Ages: {{AGE_SPOUSE_1}} / {{AGE_SPOUSE_2}}
- Target retirement age: {{RETIRE_AGE}} ({{YEARS_TO_RETIREMENT}} years out)
- Kids: {{KIDS_SUMMARY}}
- Combined gross income: {{GROSS_INCOME}}
- Tax rate (effective): {{TAX_RATE}}

## Current balances (snapshot as of {{SNAPSHOT_DATE}})
- Liquid (taxable brokerage + crypto): {{LIQUID_TOTAL}}
- Retirement (401K + IRAs): {{RETIREMENT_TOTAL}}
- 529 plans: {{COLLEGE_TOTAL}}
- Home equity (value − mortgage): {{HOME_EQUITY}}
- Total net worth: {{NET_WORTH}}

## Allocation (taxable brokerage)
{{ALLOCATION_TABLE}}
(holdings with % of liquid portfolio)

## Assumptions driving the projection
- Real return: {{REAL_RETURN}}
- Expense growth: {{EXPENSE_GROWTH}}
- 401K contribution %: {{K401_PCT}}
- IRS contribution limits 2026: $23,500/person 401K, $7,000/person IRA
- Social Security assumed @ 67: {{SS_INCLUDED}}

## Projected outcome at retirement
- Target liquid at 4% SWR: {{SWR_TARGET}}
- Projected liquid at retirement age: {{PROJECTED_LIQUID}}
- Gap (projected − target): {{GAP}}
- Implied SWR at retirement: {{IMPLIED_SWR}}

## Q-over-Q changes since last review
{{QOQ_DELTA}}
(net worth delta, notable moves in allocation, any accounts opened/closed)

## Open action items (from previous review, if any)
{{OPEN_ACTIONS}}

## Flags surfaced by the app
{{APP_FLAGS}}
(e.g. "cash position 2× emergency fund target", "tech concentration >40% of liquid", "no estate docs flag", "childcare cliff in 2028")

---

# Output format

Return ONLY the HTML body for the review panel. It will be rendered
inside this container:

<div id="qr-generated-body" style="padding:0 18px 18px">
  <!-- YOUR OUTPUT HERE -->
</div>

Structure:

1. Snapshot row — 5 tiles. Good KPIs to pick:
   - Liquid, Retirement, Net Worth, Target @ retirement, Baseline path
     (ON TRACK / WATCH / OFF TRACK)

Use the same markup as the spending review (repeat here for
convenience):

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(160px,1fr));gap:1px;background:var(--border);border-radius:4px;overflow:hidden;margin-bottom:22px">
  <div style="background:var(--bg-card);padding:14px 16px">
    <div style="font-size:10px;font-family:'JetBrains Mono',monospace;letter-spacing:.15em;color:var(--text-mute);margin-bottom:6px">KPI LABEL</div>
    <div style="font-size:20px;font-weight:700;color:var(--accent)">$XXX K</div>
    <div style="font-size:10px;color:var(--text-dim);margin-top:3px">subtitle</div>
  </div>
  <!-- 4 more tiles -->
</div>

2. Top opportunities ranked by impact — 4 to 6 cards. Each card
   should have a specific dollar-impact estimate and a concrete action
   step (not "consider X" — "do X, by Y").

<h4 style="font-size:10px;font-family:'JetBrains Mono',monospace;letter-spacing:.18em;color:var(--text-mute);text-transform:uppercase;margin:0 0 12px">Top Opportunities — Ranked by Impact</h4>
<div style="display:grid;gap:10px;margin-bottom:24px">

  <div style="background:var(--bg-card);border:1px solid var(--border);border-left:3px solid var(--accent);border-radius:4px;padding:14px 16px">
    <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:6px;flex-wrap:wrap;gap:6px">
      <span style="font-weight:600;font-size:13px">N · [Headline action]</span>
      <span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:var(--accent);background:rgba(0,229,255,.08);padding:2px 8px;border-radius:3px">+~$XXXK impact</span>
    </div>
    <p style="margin:0;font-size:12px;color:var(--text-dim);line-height:1.6">
      [2–4 sentences. Explain the mechanism. Show the math or cite the
      assumption. Acknowledge tradeoffs honestly.]
    </p>
    <div style="margin-top:10px;padding:8px 10px;background:rgba(0,229,255,.05);border-radius:3px;font-size:11px;font-family:'JetBrains Mono',monospace;color:var(--accent)">ACTION → [specific next step, deadline if applicable]</div>
  </div>

  <!-- more cards -->
</div>

Border-left color coding:
- Highest impact / aggressive growth move → var(--accent) cyan
- Tax-advantaged space → var(--accent2) green
- Risk mitigation (concentration, estate) → var(--accent3) red / var(--violet) purple
- Cash / liquidity optimization → var(--accent4) amber
- Annual flags (insurance, docs) → var(--text-mute) gray

3. The Number to Watch — closing paragraph:

<div style="background:rgba(0,229,255,.04);border:1px solid rgba(0,229,255,.15);border-radius:4px;padding:14px 16px">
  <div style="font-size:10px;font-family:'JetBrains Mono',monospace;letter-spacing:.18em;color:var(--accent);margin-bottom:8px">THE NUMBER TO WATCH</div>
  <p style="margin:0;font-size:12px;color:var(--text-dim);line-height:1.7">
    [2-3 sentences. Name the single metric that, if it moves, changes
    the retire-at-X answer most. Usually: implied SWR at retirement,
    or the gap between projected and target. Explain what would move
    it favorably.]
  </p>
</div>

4. Footer with disclaimer:

<p style="margin:16px 0 0;font-size:10px;color:var(--text-mute);font-family:'JetBrains Mono',monospace">NEXT REVIEW: {{NEXT_QUARTER_LABEL}} · Generated {{GEN_DATE}} · AI-generated analysis, not licensed financial advice — confirm tax strategy and investment changes with a CPA or fiduciary before acting.</p>

---

# Tone

- Advisor-grade, not sycophantic. Call out real risks without
  catastrophizing. Celebrate real wins without inflating.
- Specific over general. "Roll Hadley's $32K traditional IRA into her
  401K to enable backdoor Roth" beats "consider Roth optimization."
- Quantify impact. Every action card should attach a ballpark dollar
  figure at retirement age, even if it requires stating assumptions.
- Acknowledge aggressive posture if user has opted into it — don't
  reflexively recommend "diversification" against an intentional
  concentrated bet. But flag the downside number so it's eyes-open.
- End on the single metric most worth tracking. Don't repeat the
  whole review.
```
