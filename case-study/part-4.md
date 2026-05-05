# Part 4: Rollout & Measurement

**The brief:** *"Create an action plan on how you'd spend your time on this project in your first 30 days. How would you pilot the system and measure whether it is working?"*

**The plan, in one paragraph:** Week 1 = talk with the sales team and audit the data; Week 2 = build the system end-to-end and test it myself; Week 3 = first Monday goes live with the pilot territory and iterate on SDR feedback; Week 4 = system ready, onboard remaining SDRs, lock the dashboard. Measure on **5 KPIs** (scored→meeting conversion, reply rate, sequencing rate as adoption proxy, coverage %, pipeline→closed-won) tracked via a Looker dashboard pulling from Snowflake + Salesforce. Wire **4 feedback loops** from day 1 so the system learns. Phase 2 (after 90 days of pilot data) replaces the heuristic weights from Part 1 with empirical ones via logistic regression on signal-firing → won/lost outcomes — the data flywheel.

---

## 30-day plan

### Week 1 — Talk to the sales team
- **Conversations with the sales team:** how do they cut territory today? How do they run outbound — what works, what's broken? Where do they spend Monday morning?
- **Audit where the data sits:** Salesforce object model, Clay workflows already in production, Snowflake schemas, Amplemarket sequence setup, Attention's marketing analytics
- **Walk the team through the project** to validate the approach early — surfaces objections + buy-in before any code is written
- **Lock pilot scope** (1 territory, 2 SDRs) and **kill criteria** with sales leadership

### Week 2 — Build & test myself
- Wire Stage 1 — signal collection across 10+ APIs (Clay, RB2B, GA4, LinkedIn Campaign Manager, Reddit Ads, Common Room, Crunchbase, G2, Amplemarket, SF)
- Wire Stage 2 — scoring math + 2×2 routing matrix
- Wire Stage 3 — LLM Hook Agent + Amplemarket templated cadence connections + Apps Script Sheet integration
- Build the cadence library in Amplemarket (10 templated sequences with custom-field tokens) + the AI hook prompt
- Test end-to-end on a 100-account sample before going live

### Week 3 — First real-time test
- **Monday:** full pipeline runs on the pilot territory for the first time
- Daily 15-min standup with pilot SDRs to surface bugs + qualitative friction
- Daily monitoring: signal volume per source, false-positive rate, system uptime, sequencing rate
- Iterate in real-time: tune signals, refine the hook prompt, adjust cadence routing
- Friday: first weekly performance report

### Week 4 — System ready
- Pipeline running smoothly, no critical bugs
- Onboard remaining SDRs in the pilot territory
- Lock the measurement dashboard in Looker (or Mode) → Snowflake + Salesforce
- Decision point: scale to the next territory or extend the pilot

---

## Measurement — 5 KPIs

| KPI | What it tells us |
|---|---|
| **Scored accounts → meetings booked** | Overall system conversion. The most direct measure of "is this scoring + outreach working?" |
| **Reply rate** (per cadence, per signal) | Message resonance + signal quality. Per-cadence breakdown shows which templates land; per-signal breakdown shows which signals predict actual replies. |
| **Sequencing rate** (% of Tier 1 actually actioned by SDRs) | Tool adoption. If this drops, SDRs are losing trust in the system or the hooks. Most important leading indicator. |
| **Coverage %** (% of accounts firing ≥1 signal weekly) | Pipeline health. Tells us whether signal collection is working at scale. |
| **Pipeline → closed-won** (90-day cohort) | Revenue impact. Lagging metric, but the only one that matters for the business case. |

**Tracking infrastructure:** Looker (or Mode) dashboard refreshed daily, pulling from:
- **Snowflake** — signal-firing history, scoring history, account states week-over-week
- **Salesforce** — sequencing actions, replies, opps created, closed-won
- **Amplemarket** — sequence performance, reply data

One dashboard, one source-of-truth view. Salesforce-native reports for adoption metrics that are SF-only (sequencing rate per SDR, etc.).

---

## Feedback loops — wired from day 1

1. **Cadence-level** — per-cadence reply rate + meeting rate tracked weekly via Amplemarket webhooks → Snowflake. A/B test variants. Weak performers (<5% reply) rewritten or retired. Winning patterns flow back into the AI hook prompt.
2. **Agent-level** — LLM hook generator: track replies per hook style → prompt tuning. Sub-type classifier: weekly accuracy spot-check. Orchestrator: error log per stage → harden weak API integrations.
3. **Outcome-back-to-source** — When a deal closes-won, the AE tags the signal that drove first interest in SF (required field on Closed-Won stage transition). This trains the Phase 2 logistic regression and tells us empirically which signals predict revenue.
4. **Human** — Weekly 30-min SDR check-in (qualitative ground truth — what worked, what didn't) · Monthly review with sales leadership (metrics + cadence performance) · Quarterly CEO/CRO review (ROI + expansion).

---

## Phase 2 — improve scoring with past + ongoing data

The Part 1 weights are educated estimates from industry research. They're a starting point, not the end state.

### What changes after 90 days of pilot data

1. **Backtest** the heuristic weights against Attention's actual closed-won and closed-lost deals from the pilot. Reconstruct signal-firing history at the time of first SDR touch for every deal.
2. **Run a logistic regression** on `(signals fired) → (deal won/lost)` to learn empirical weights from real conversion patterns.
3. **Validate against a hold-out set** to ensure weights generalize beyond the training period.
4. **Replace** the heuristic Part 1 weights with the empirical ones. Refresh quarterly as more outcomes accumulate.

### Beyond v2

- **Per-industry weight tuning** — different industries respond to different signals. Tune weights by vertical once we have enough data per segment.
- **AE feedback loop** — when an AE closes a deal, capture which signal sequence drove it. Feed back into routing logic.
- **The data flywheel:** more deals → better weights → better scoring → better deals. The system gets smarter every week.

---

## Pilot decision criteria (locked Week 1 with sales leadership)

- **Reply rate beats the existing manual outbound process by ≥30% over 4 weeks** → roll out to all SDRs in weeks 5–6
- **Inconclusive** → extend pilot 4 more weeks, recalibrate based on SDR feedback
- **Pilot underperforms** → root-cause first (signal quality? cadence body? hook quality? system bugs?), don't abandon. Fix the pinch point.

**Hard kill criteria:**
- Sequencing rate <40% after 4 weeks (SDRs aren't trusting the suggestions) → reassess hook + cadence quality before continuing
- System fails to run reliably 2+ Mondays in a row → infra audit before continuing
