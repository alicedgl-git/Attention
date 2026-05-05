# Part 1: Buying Signals

Ten signals tracked weekly across the 50k ICP universe, organized into three categories. Each carries a weight on a 100-point scale. An account's **Intent Score** = sum of weights from signals fired this week. Scoring formula and tier routing: see Part 2.

## Why three categories

| Category | What it captures | Why we rank it where we do |
|---|---|---|
| **First-party intent** | Where the buyer shows themselves to us — visiting our site, engaging with our ads, attending our webinars, posting in communities we monitor | **Highest-weighted by industry consensus.** Pocus, UnifyGTM, RB2B research: "first-party signals should always be the highest-weighted inputs in your scoring model — a prospect visiting your pricing page is *telling you* something definitive." Lower volume than third-party signals, but materially higher conviction per fire. |
| **Trigger events** | Where buying windows open — new leadership, scaling teams, public pain | **Predict timing.** Executive change is the most-cited "buying window" signal; sales-team-scaling correlates with ramp pain that Attention solves. Strong but inferential. |
| **State / displacement / budget** | Where deal economics work — competitor entrenched, competitor weakening, capital available | **Inferential context.** Tells us *whether* a deal is viable, not *when* to reach. Lowest weights — these are compounders, not triggers. |

## The 10 signals

### First-party intent (weight 46/100)

| # | Signal | Type | Example trigger | Why it matters | Weight | Monthly volume | Data source |
|---|---|---|---|---|---|---|---|
| 1 | **Website visits** | P | 2+ visits to attention.com in last 30 days, including the demo page OR ≥2 of {/product, /solution, /integrations, /customers, /pricing} pages, with persona-matched deanon | The single highest-conviction signal class for signal-led GTM (Pocus, UnifyGTM consensus). With strict page-filtering + persona-matched deanon, a multi-visit pattern means the buyer is actively evaluating us specifically — past research, into shortlisting. Volume is naturally low because the strict criteria filter out junk traffic, but conviction-per-fire is the highest in the table. | **20** | ~300–800/mo | RB2B / Clearbit Reveal / 6sense (visitor deanonymization) · GA4 events on /demo, /pricing, /product, /integrations, /customers, /solution → ingested via Clay → Snowflake |
| 2 | **Content engagement** | P | Webinar registrant or attendee, podcast listener, in-person event attendee, downloaded a gated asset, or actively engaged (like / comment / share) with Attention's marketing content in last 30 days | Webinar attendance is one of the strongest digital B2B intent signals — typical 20–40% attendee-to-qualified-lead conversion (industry research). A buyer carving out 45 minutes for our content is showing a level of intentional engagement almost no other channel matches. | **14** | ~500–1,500/mo | Attention's marketing automation (HubSpot / Customer.io / Marketo) · webinar attendee exports (Goldcast / Demio / Zoom Webinar) · LinkedIn Page engagement export · Spotify / Apple podcast download tracking · Salesforce campaign membership |
| 3 | **Ad interaction** | P | ICP-role person engages with Attention's LinkedIn or Reddit ads in last 30 days — click-through, video view ≥25%, comment, save/share, or lead-gen form open (without submission) | "The gap between conversion and engagement is only two days." Engaging with an ad — clicking, watching, commenting — is now treated as a near-equivalent intent signal to a form fill (just earlier in the funnel). Intent-driven outreach off engagement signals lifts conversion up to 93%. Especially strong because the ad already targeted the ICP — the engagement doubles as an ICP-confirmation. | **8** | ~500–1,500/mo (depends on Attention's ad spend) | LinkedIn Campaign Manager API (engagement events: clicks, video views, comments, lead-gen form opens) · Reddit Ads Manager API · Clay enrichment to resolve ad engagers to companies and ICP roles → Snowflake |
| 4 | **Slack community members & messages** | P | ICP-role person joins or actively posts in target communities (Pavilion, RevGenius, Modern Sales Pros, Sales Hacker, RevOps Co-op) — especially if the post discusses pain points (call coaching, forecasting, ramp, AI sales tools) | Community engagement precedes purchase intent — Common Room's foundational thesis, validated by Apollo's acquisition of Pocus and the broader signal-based selling category. Community signals fire 3–6 weeks earlier than most other public buying signals. Low volume after ICP filtering, but exceptionally early. | **4** | ~100–300/mo (filtered to ICP roles + topical relevance) | Common Room (Slack / Discord / Discourse / X / GitHub community signal capture) · Champify-style new-member alerts · manual community monitoring as backup → Snowflake |

### Trigger events (weight 34/100)

| # | Signal | Type | Example trigger | Why it matters | Weight | Monthly volume | Data source |
|---|---|---|---|---|---|---|---|
| 5 | **Executive change / Key role job posts** | P | VP Sales / CRO / Head of Sales / Head of RevOps / RevOps Lead / Sales Enablement / Sales Ops / Sales Strategy joined or got promoted in the last 90 days, OR a job posting for these roles is open | New leaders re-evaluate the entire sales tech stack in their first 90 days. Avg VP Sales tenure ≈ 18 months and they have ~1 quarter to prove themselves. Single highest-lift trigger for sales-tooling purchase. The role-posting variant captures buying capacity opening up before the person is hired. | **12** | ~2,500–3,500/mo | LinkedIn Sales Navigator job-change tracking (Clay) · LinkedIn Jobs (recent postings) · Salesforce contact title-change history · TechCrunch / PR Newswire APIs · Amplemarket enrichment |
| 6 | **Sales team scaling** | C | Aggressive AE / SDR / CSM / AM hiring velocity: 3+ open roles posted in last 30 days | Active scaling = ramp pain, coaching pain, forecast pain — exactly Attention's wedge. Rate-of-change beats absolute team size: a team going 20 → 35 reps signals an acute coaching problem incoming in 60–90 days. Volumetrically high so most useful as a compounder with #5. | **11** | ~5,000–7,000 in active state | LinkedIn Jobs · Greenhouse / Lever ATS scrapers via Clay · Amplemarket enrichment |
| 7 | **Buyer pain / ICP topical engagement** | P | VP Sales / CRO / RevOps posts publicly (LinkedIn, podcast, interview, earnings call) about pipeline pain, forecast misses, ramp problems, coaching gaps, or AI sales tooling — pain-coded or neutral topical | Direct verbalized intent from the actual buyer. Pain-coded posts (e.g., "we missed forecast last quarter") give the AI hook a direct quote-back; neutral topical posts still lift response rates ~32% (Landbase research). Cadence library routes by sub-type: pain-coded → "Empathy + Solution"; neutral → "Topical Hook." | **11** | ~1,500–3,000/mo | LinkedIn social monitoring via Clay (sentiment classifier flags pain language) · Common Room · Listen Notes API for podcast transcripts · Quartr / AlphaSense for public earnings-call transcripts |

### State / displacement / budget (weight 20/100)

| # | Signal | Type | Example trigger | Why it matters | Weight | Monthly volume | Data source |
|---|---|---|---|---|---|---|---|
| 8 | **Competitor usage** | C | Job posts requiring experience with Gong / Chorus / Clari / Outreach / Salesloft | Proves the company is actively using a competitor AND scaling on top of that stack. Pure displacement opportunity — shorter cycles, higher win rate than category-education sells. | **7** | ~3,000/mo unique accounts | LinkedIn Jobs · Greenhouse / Lever scrapers via Clay |
| 9 | **Competitor churn window** | C | G2 review-rating drop on a competitor the account uses; layoffs at competitor (Gong, Chorus, Clari, Sybill); known contract-end window | An active buying window is opening regardless of our outreach. Industry data: G2-flagged deals run ~2× larger and competitor layoffs put "every vendor under review." Rare but high conviction when it fires. | **7** | ~50–200/mo | G2 Buyer Intent API · layoffs.fyi · LinkedIn employee-count tracking on competitor pages · Crunchbase News · win/loss interview notes (custom SF field) · Amplemarket signal feeds |
| 10 | **Recent funding** | C | Raised a new round in the last 6 months (Series A through pre-IPO) | Budget unlocked + board pressure to deploy capital toward revenue acceleration. Necessary-but-not-sufficient signal. | **6** | ~400–600/mo new triggers (~2,500 in 6-mo state) | Crunchbase API · PitchBook (Clay) · TechCrunch / Axios Pro Rata feeds · SEC EDGAR for late-stage / pre-IPO · Amplemarket enrichment |

**Total weight: 100**

---

## Why first-party intent earns the top weight (defending the ranking)

This case is run by signal-led GTM thinking — not generic ABM. Three pieces of consensus from the research:

1. **Pocus / UnifyGTM:** "For teams running a signal-based selling motion, first-party signals should always be the highest-weighted inputs in your scoring model. A prospect visiting your pricing page three times in a week is telling you something definitive."
2. **B2B SaaS conversion benchmarks (2026):** Top-decile teams convert 8–15% of website visitors to lead — first-party engagement is the highest-fidelity signal class available. Webinar-attendee-to-qualified-lead conversion is 20–40%.
3. **LinkedIn ads as intent (Dreamdata 2026):** "The gap between conversion and engagement is only two days... a video view, comment, or click that didn't convert is just earlier in the funnel, not less interested."

Trigger events (#5–#7) come second because they predict *timing*. State signals (#8–#10) come last because they describe *context*, not the moment.

The hidden multiplier: first-party signals **arrive with personalization material baked in**. We know which page they visited, which webinar they attended, which ad they engaged with. The AI hook can reference the exact behavior — meaningfully outperforming "Hi {first_name}, saw your company raised a Series B."

---

## Assumptions

Three explicit assumptions, all transparent enough to be redlined:

1. **First-party-led ranking assumes Attention has meaningful inbound volume in the ICP.** If Attention's content marketing and ad spend produce <100 ICP-relevant first-party fires per month, the weight on signals #1–#4 should drop and trigger events should rise. This is testable in the first 30 days of pilot data.
2. **Volume estimates are order-of-magnitude.** Each "monthly volume" uses base rates × ICP filter × rolling-window math. Real numbers will differ by up to 2× until we measure for a quarter.
3. **Positive-only scoring.** Suppression-eligible accounts (active opps, current customers, recent layoffs at the prospect, churned <6mo, already-engaged in Amplemarket last 90d) are *filtered* before scoring runs — never penalized inside the score. See Part 3 hard exclusion filters.

---

## Phase 2 — calibrating weights with past data

The cleanest way to validate these weights is to backtest against Attention's closed-won and closed-lost deals over the last 12 months. With Salesforce + Snowflake access:

1. **Reconstruct signal-firing history** at the time of first SDR touch for every deal in the period — pulling LinkedIn / Crunchbase / web-analytics / LinkedIn-Ads / Common-Room data retroactively, timestamped.
2. **Run a logistic regression** on `(signals fired) → (deal won/lost)` to learn empirical weights from actual conversion data. Specifically interesting: does the first-party-top weighting hold for Attention's actual deals, or do trigger events outperform?
3. **Validate against a hold-out set** to ensure weights generalize beyond the training period.
4. **Refresh weights quarterly** as the model retrains.

This is **not part of the v1 ask** — Attention's past data isn't available for this exercise — but it's the natural Phase 2. The weights here are an opinionated starting point grounded in best-in-class GTM research; real conversion data is the only way to fully calibrate.
