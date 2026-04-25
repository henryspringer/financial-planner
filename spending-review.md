# Quarterly Spending Review — prompt template

This template is consumed by the "Generate Quarterly Review" button on the Spending tab. The app substitutes `{{placeholders}}` with live data from localStorage at click time, copies the result to the user's clipboard, and the user pastes it into their AI agent.

The agent returns HTML matching the schema below, which the user pastes back into the modal. The app saves it to localStorage under `planner_qr_spending_{{QUARTER_ID}}` and renders it in the panel.

---

## Prompt (with substitutions)

```
You are a financial analyst producing a quarterly spending review for a
household using the 11-category budget framework in the attached
personal finance dashboard.

Do not invent numbers. Use only the data provided below. If a
category's variance is small (under ±10%), note it briefly — don't
fabricate a narrative for it.

## The quarter
{{QUARTER_LABEL}} — covers {{MONTH_START}} through {{MONTH_END}}

## Monthly totals (checking + credit card combined)
{{MONTHLY_TOTALS_TABLE}}

Quarterly total: {{Q_TOTAL}}
Quarterly budget target: {{Q_BUDGET}}
Variance: {{Q_VARIANCE_PCT}}

## Category performance — Q avg/mo vs budget
{{CATEGORY_TABLE}}
(columns: category, Q avg/mo, budget, variance %, variance $)

## Notable large transactions
{{OUTLIER_LIST}}
(anything ≥ $2,000 in a single txn, or the top 5 merchants by spend)

## Rolling 12-month context
{{ROLLING_12M_SUMMARY}}
(avg/mo by category vs budget, so the agent can tell quarterly
 anomalies apart from persistent patterns)

## Household context (user-provided at setup)
{{USER_CONTEXT}}
(e.g. "3 kids under 3, Henry's W2 is primary income, one-time kitchen
 reno closing out in Q1, aspirational $12K/mo budget")

---

# Output format

Return ONLY the HTML body for the review panel. It will be rendered
inside this container:

<div id="sqr-generated-body" style="padding:0 18px 18px">
  <!-- YOUR OUTPUT HERE -->
</div>

Structure:

1. Snapshot row — 5 KPI tiles in a grid. Use this markup:

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(160px,1fr));gap:1px;background:var(--border);border-radius:4px;overflow:hidden;margin-bottom:22px">
  <div style="background:var(--bg-card);padding:14px 16px">
    <div style="font-size:10px;font-family:'JetBrains Mono',monospace;letter-spacing:.15em;color:var(--text-mute);margin-bottom:6px">KPI LABEL</div>
    <div style="font-size:20px;font-weight:700;color:var(--violet)">$XX,XXX</div>
    <div style="font-size:10px;color:var(--text-dim);margin-top:3px">subtitle</div>
  </div>
  <!-- 4 more tiles -->
</div>

Good KPIs to pick (in priority order): Q total, Avg/mo, Ex-one-offs
normalized /mo, Biggest over-budget category, Biggest under-budget
category, Verdict (one of: ON TRACK / WATCH / OFF TRACK).

2. Category performance cards — 3 to 5 cards ranked by variance
magnitude (absolute $, not %). Use this markup:

<h4 style="font-size:10px;font-family:'JetBrains Mono',monospace;letter-spacing:.18em;color:var(--text-mute);text-transform:uppercase;margin:0 0 12px">Category Performance</h4>
<div style="display:grid;gap:10px;margin-bottom:24px">

  <div style="background:var(--bg-card);border:1px solid var(--border);border-left:3px solid var(--accent);border-radius:4px;padding:14px 16px">
    <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:6px;flex-wrap:wrap;gap:6px">
      <span style="font-weight:600;font-size:13px">[Category] — $X,XXX/mo vs $X,XXX budget</span>
      <span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:var(--accent);background:rgba(0,229,255,.08);padding:2px 8px;border-radius:3px">±XX%</span>
    </div>
    <p style="margin:0;font-size:12px;color:var(--text-dim);line-height:1.6">[Narrative — why this variance, is it one-off or structural, what's driving it]</p>
    <div style="margin-top:10px;padding:8px 10px;background:rgba(0,229,255,.05);border-radius:3px;font-size:11px;font-family:'JetBrains Mono',monospace;color:var(--accent)">WATCH → [one-line action or observation]</div>
  </div>

  <!-- more cards -->
</div>

Card border-left colors by verdict:
- Over budget, structural concern → var(--accent3) red
- Over budget, one-off / expected → var(--accent4) amber
- Under budget, positive → var(--accent2) green
- Neutral / noteworthy → var(--violet) purple
- Minor / fyi → var(--text-mute) gray

3. The Number to Watch — closing paragraph in this container:

<div style="background:rgba(0,229,255,.04);border:1px solid rgba(0,229,255,.15);border-radius:4px;padding:14px 16px">
  <div style="font-size:10px;font-family:'JetBrains Mono',monospace;letter-spacing:.18em;color:var(--accent);margin-bottom:8px">THE NUMBER TO WATCH</div>
  <p style="margin:0;font-size:12px;color:var(--text-dim);line-height:1.7">
    [2-3 sentences. The single most important thing to track going into
    next quarter. If the quarter was distorted by one-offs, state the
    normalized baseline clearly. If a structural trend is emerging,
    name it.]
  </p>
</div>

4. Footer:

<p style="margin:16px 0 0;font-size:10px;color:var(--text-mute);font-family:'JetBrains Mono',monospace">NEXT REVIEW: {{NEXT_QUARTER_LABEL}} · Generated {{GEN_DATE}}</p>

---

# Tone

- Direct. No filler. No disclaimers.
- If discipline is real (multiple categories under budget), say so.
- If overspending is structural not behavioral (one-time house
  project, annual tax payment hitting one month), distinguish that
  explicitly — don't let one-off variance read as a budget failure.
- If a variance is benign (travel under in Q1 because travel is always
  under in Q1), don't pretend it's notable.
- End on the single most actionable insight.
```
