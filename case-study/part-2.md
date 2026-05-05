# Part 2: Scoring & Tiering

**The brief:** *"Propose a method for scoring accounts by their propensity to buy and how strong of a fit they are for Attention."*

**The method, in one paragraph:** Two scores per account, both out of 100. **Intent Score** (propensity to buy) sums the 10 weighted signals from Part 1 plus a multi-signal bonus. **Fit Score** (Attention-deal viability + ACV potential) is a firmographic match across industry, sales motion intensity, and growth stage. Each gets a binary High/Low band. Plot accounts on a **2×2 matrix** and route into 4 tiers: SDR (high Intent + high Fit), Auto-outbound (high Intent + low Fit), Watchlist (low Intent + high Fit, re-scored weekly), Skip (low Intent + low Fit). A multi-signal bonus rewards convergence.

---

## Intent Score (out of 100, weekly refresh)

```
intent_score = Σ (signal_weight if fired) + multi_signal_bonus
```

**Signal weights** (from Part 1, summing to 100):

| Category | Signal | Weight |
|---|---|---|
| First-party intent | 1. Website visits | 20 |
| First-party intent | 2. Content engagement | 14 |
| First-party intent | 3. Ad interaction | 8 |
| First-party intent | 4. Slack community | 4 |
| Trigger event | 5. Executive change / Key role job posts | 12 |
| Trigger event | 6. Sales team scaling | 11 |
| Trigger event | 7. Buyer pain / ICP topical engagement | 11 |
| State / displacement | 8. Competitor usage | 7 |
| State / displacement | 9. Competitor churn window | 7 |
| State / budget | 10. Recent funding | 6 |

**Multi-signal bonus** (composite signals are far more predictive than any single one — 6sense + Bombora research shows ~20% lift in pipeline-conversion predictiveness when signals are stacked):

| Distinct signals firing | Bonus |
|---|---|
| 1 | +0 |
| 2 | +5 |
| 3 | +12 |
| 4+ | +25 |

**Intent bands:**
- **High Intent: ≥ 30**
- **Low Intent: < 30**

What gets you to High Intent: 2 first-party signals (Website visits 20 + Content engagement 14 = 34 + 5 = 39 ✓), or 1 first-party + 1 trigger (Website visits 20 + Executive change 12 = 32 + 5 = 37 ✓), or 3 trigger signals (~34 + 12 = 46 ✓). Single signal alone ≠ enough.

---

## Fit Score (out of 100, monthly refresh)

The ICP filter (VP Sales / CRO / RevOps + 50+ employees + 6+ sales reps) is a hard gate upstream — accounts failing it aren't in the 50k pool. Fit Score differentiates *within* the qualified universe by **how big a deal Attention can land** (Fit ≈ ACV potential).

| Component | Weight | Rule |
|---|---|---|
| **Industry tier** | 50 | Tier 1 (high-velocity SaaS, fintech, modern B2B): **50** / Tier 2 (B2B services, healthtech, prof. services): **30** / Tier 3 (passes ICP but lower fit — logistics, manufacturing, traditional B2B): **15** |
| **Sales motion intensity** | 30 | Sales reps ÷ total employees. Sales-led (≥10% reps OR 2+ active SDR/BDR roles): **30** / Hybrid (5–10%, mixed inbound/outbound): **20** / Sales-light (<5%, PLG or service-led): **5** |
| **Growth / funding stage** | 20 | Series B+ in last 24mo OR known >$50M ARR OR public + growing: **20** / Series A or established mid-market: **12** / Bootstrapped, pre-seed/seed, declining, or unclear: **5** |

**Fit bands:**
- **High Fit: ≥ 70**
- **Low Fit: < 70**

Industry tier carries the most weight because it's the strongest predictor of ACV. Sales motion intensity matters because Attention's value scales linearly with calls per rep — a sales-led company with 80 reps is a much bigger deal than a PLG company with 8.

---

## The 2×2 routing matrix → 4 tiers

|              | **High Intent (≥30)** | **Low Intent (<30)** |
|---|---|---|
| **High Fit (≥70)** | 🟢 **Tier 1 — SDR** | 🔵 **Tier 3 — Watchlist** |
| **Low Fit (<70)**  | 🟡 **Tier 2 — Auto-outbound** | ⚪ **Tier 4 — Skip** |

| Tier | What happens | Estimated weekly volume |
|---|---|---|
| **Tier 1 — SDR** | Hand-personalized outreach via Amplemarket. Strongest signal determines the cadence (Part 3 playbook). AI generates the first hook from signal evidence. | ~250/wk (5 SDRs × 50 capacity) |
| **Tier 2 — Auto-outbound** | High intent but smaller deal size. Programmatic Amplemarket sequence with templated openers. Zero per-touch cost. | ~2,000/wk |
| **Tier 3 — Watchlist** | Perfect-fit account, no intent firing this week. **Don't pitch — but re-score every Monday.** The moment any signal fires, jumps to Tier 1. Operationally: SDR sees them in the Lead View under "Watchlist" tab for context, not action. | ~1,500/wk (rotating) |
| **Tier 4 — Skip** | No fit, no intent. Don't touch. Re-evaluated weekly — a signal firing can pull them to Tier 2. | ~6,000+/wk |

**Why Watchlist matters:** without it, perfect-fit accounts disappear from view until they happen to fire intent at a moment we're paying attention. With it, they're held in a queue — operationally costless (no outreach) but strategically valuable (instant pickup when intent fires).

---

## Worked examples

### Tier 1 — Acme Sales Co. (High Fit × High Intent)

**Fit Score:** B2B SaaS (Tier 1 industry, 50) + 80 reps / 600 employees = 13% sales density (sales-led, 30) + Series C raised 4mo ago (20) = **100 → High Fit** ✓

**Intent Score:**

| Signal | Weight |
|---|---|
| Website visits — multi-visit attention.com incl. /demo + /pricing | 20 |
| Content engagement — VP Sales attended last week's webinar | 14 |
| Executive change — VP Sales hired 45d ago | 12 |
| Sales team scaling — 4 AE + 1 SDR roles posted last 30d | 11 |
| Competitor usage — hiring AEs requiring Gong | 7 |

Sum = 64, +25 bonus (5 signals) = **89 → High Intent** ✓

**Routing:** **Tier 1 — SDR.** Strongest signal = Website visits → cadence routes to "Personalized · AI hook ready."

### Tier 2 — Beta Systems Inc. (Low Fit × High Intent → Auto)

**Fit Score:** Logistics software (Tier 3 industry, 15) + 6 reps / 140 employees = 4% (sales-light, 5) + bootstrapped (5) = **25 → Low Fit**

**Intent Score:** Executive change (12) + Competitor usage (7) + Sales scaling (11) = 30 + 12 bonus (3 signals) = **42 → High Intent** ✓

**Routing:** **Tier 2 — Auto-outbound.** Intent is firing but the deal economics don't justify SDR time. Goes to programmatic Amplemarket with templated opener. Reply rate floor is lower; cost-per-touch is ~zero.

### Tier 3 — Gamma Corp. (High Fit × Low Intent → Watchlist)

**Fit Score:** Mid-market SaaS (Tier 1, 50) + 110 reps / 950 employees = 12% (sales-led, 30) + IPO'd 18mo ago (20) = **100 → High Fit** ✓

**Intent Score:** No signals firing this week → **0 → Low Intent**

**Routing:** **Tier 3 — Watchlist.** Not pitched this week. Re-scored next Monday. The moment a signal fires (a new VP hire, a funding round, a website visit), Gamma jumps to Tier 1 — instant action without losing track of the account in the meantime.

---

## Phase 2 — calibration with past data

The weights here are educated estimates from industry research (Pocus / UnifyGTM / 6sense / Bombora). The natural maturation step: backtest against Attention's closed-won and closed-lost deals over 12 months using logistic regression on `(signals fired) → (won/lost)` to learn empirical weights from actual conversion data. Not in v1 scope — Attention's past data isn't available for this exercise — but the highest-leverage step once v1 is live. Refresh quarterly.
