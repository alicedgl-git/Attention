---
name: Attention case study — full session memory
description: Complete context snapshot from the GTM Engineering case study prep — project background, all final design decisions, file locations, user preferences, and outstanding work
type: project
originSessionId: 84048f24-d0f4-45f2-ac72-70c675df464b
---
# Attention GTM Engineering Case Study — Memory Snapshot

**Last updated:** 2026-05-05
**Working directory:** `/Users/alicedoglioli/Desktop/Attention/case-study/`

---

## Project context

- **User:** Alice Doglioli (currently at Concord — email `marketing@concordnow.com`)
- **Applying to:** Attention (attention.com — AI sales-call assistant; competitors include Gong, Chorus / ZoomInfo, Clari Copilot, Sybill, MeetRecord, Avoma, Modjo)
- **Role:** Growth Engineering
- **Stage:** Final-round case study presentation, 45 minutes
- **Brief:** 4 parts — Signals · Scoring · System Design + GTM · Rollout & Measurement
- **Tech stack constraint** (from brief): Salesforce · Clay · Snowflake · Amplemarket · Attention

**Framing note:** Alice is presenting the architecture as her own design thinking. No references to anything she may have read or heard about Attention's internal tooling.

## ICP (locked, used as binary filter)
- VP Sales, CRO, or RevOps leaders
- Companies with 50+ employees
- Sales teams of 6+ reps
- Target universe: ~50k ICP accounts after filtering from ~1M

---

## Files (all under `/Users/alicedoglioli/Desktop/Attention/case-study/`)

| File | Role |
|---|---|
| `part-1.md` | Signals (10 signals, weights summing to 100, sources, rationale, Phase 2 calibration) |
| `part-2.md` | Scoring & Tiering (Fit × Intent matrix, 4 tiers) |
| `part-3.md` | System architecture (Claude Code orchestrator + 3 Python stages) |
| `part-4.md` | 30-day rollout + 5 KPIs + 4 feedback loops + Phase 2 |
| `system-architecture.html` | Visual companion for Part 3 (Mermaid + agent cards + SDR mocks) |
| `signals-table.html` | Copy-paste signal table for Google Docs |
| `presentation.html` | **7-slide deck** (cover + 6 content slides) |
| `research-notes.md` | Citations from web research |

---

## Final design — Part 1 (10 signals, weights sum to 100)

Three categories. Em-dashes purged from cells (descriptions are plain sentences). Source column = ONE tool per signal.

**First-party intent (46/100):**
1. Website visits — P — w. 20 — vol ~100 to 400/mo — **RB2B**
2. Content engagement — P — w. 14 — vol ~200 to 800/mo — **Goldcast**
3. Ad interaction — P — w. 8 — vol ~200 to 800/mo — **LinkedIn Campaign Manager**
4. Slack community — P — w. 4 — vol ~100 to 300/mo — **Amplemarket** (Alice's choice; Common Room is the natural alternative if Amplemarket doesn't natively cover community discussions)

**Trigger events (34/100):**
5. Executive change / Key role — P — w. 12 — vol ~2,500 to 3,500/mo — **Amplemarket**
6. Sales team scaling — C — w. 11 — vol ~3,000 to 5,000 — **Amplemarket**
7. Buyer pain / topical engagement — P — w. 11 — vol ~500 to 1,500/mo — **LinkedIn (Sales Nav)**

**State / displacement / budget (20/100):**
8. Competitor usage — C — w. 7 — vol ~1,500 to 2,500/mo — **Amplemarket**
9. Competitor churn window — C — w. 7 — vol ~50 to 200/mo — **G2**
10. Recent funding — C — w. 6 — vol ~200 to 400/mo — **Crunchbase**

**Phase 2:** backtest weights against closed-won/lost data via logistic regression after 90 days.

**Volumes are order-of-magnitude estimates** — Alice flagged earlier numbers were too high; revised conservatively.

---

## Final design — Part 2 (scoring & tiering)

**Intent Score** (weekly, /100): `Σ signal weights + multi-signal bonus`. Bonus: 2 sigs = +5, 3 sigs = +12, 4+ sigs = +25. Bands: High ≥ 30, Low < 30.

**Fit Score** (monthly, /100):
- Industry tier (50 pts): SaaS/fintech 50 / B2B services 30 / others 15
- Sales motion intensity (30 pts): sales-led ≥10% reps 30 / hybrid 20 / sales-light 5
- Growth/funding stage (20 pts): Series B+ or >$50M ARR 20 / Series A 12 / earlier 5
- Bands: High ≥ 70, Low < 70

**2×2 routing matrix → 4 tiers:**

| | High Intent (≥30) | Low Intent (<30) |
|---|---|---|
| **High Fit (≥70)** | Tier 1 — SDR personalized | Tier 3 — Watchlist (re-scored Mon) |
| **Low Fit (<70)** | Tier 2 — Auto-outbound | Tier 4 — Skip |

### KILLED (do not bring back without explicit ask)

- ❌ **Velocity flag (🔥 hot account)** — Alice dismissed: "hard to put in place and not useful." All references purged from all files.
- ❌ **S+ tier with auto-Tier-1 override** — earlier iteration; Alice pushed back.

---

## Final design — Part 3 (system architecture)

**Schedule:** runs every Monday at 6 AM. End-to-end < 30 min. SDRs open Sheet at 8 AM with everything ready.

**Honest agent count:** **2.** Architecture: 1 Claude Code orchestrator + 3 Python pipeline stages + 1 LLM Hook Agent (embedded in Stage 3).

**Stage 1 — Signal Collector (Python ETL):** calls 10+ APIs in parallel. 2 inline LLM micro-calls for sub-type classification on signals #5 (senior vs operational) and #7 (pain vs topical). Writes to Snowflake (append-only signal store).

**Stage 2 — Scorer (pure Python):** reads Snowflake + SF metadata. Fit + Intent + multi-signal bonus → 2×2 matrix → Tier 1/2/3/4. Writes scores to SF.

**Stage 3 — Distributor (Python + LLM Hook Agent):** owner-driven workflow (NOT signal-type-driven).

### Cadence library — owner-driven (CRITICAL design decision)

- **Tier 1 (SDR-owned):** Cadence cell shows "Personalized — AI hook ready." Stage 3 LLM Hook Agent generates 1-2 sentence opener per Tier 1 account during Monday batch (~250 calls/week, ~$0.001 each, retry + brand-safety QC). Hook stored in **SF custom field `ai_hook`**. SDR marks Yes in Sheet → Apps Script creates Amplemarket draft sequence (hook pre-filled in step 1 + 5-touch templated structure for steps 2-5) → SDR composes the email in their own voice → sends. ~10-15 min/account.
- **Tier 2 (Auto-outbound):** Cadence cell shows matched templated cadence name. 10 templated Amplemarket sequences (one per signal) with custom-field tokens populated by Stage 3 Python (e.g., `{{webinar_name}}`, `{{competitor_name}}`). SDR marks Yes (or overrides cadence dropdown) → Apps Script pushes contact → fires automatically. **No LLM in Tier 2 path.** ~30 sec/account.

### KILLED (do not bring back)

- ❌ **AI-assisted full sequence drafting** (LLM drafts all 5 emails) — Alice dismissed as overkill; replaced with simpler AI-hook-only-for-Tier-1.
- ❌ **Per-contact AI hooks inside templated Tier 2 cadences** — replaced with custom-field tokens. Generic body, deterministic data.
- ❌ **"Surface 1 / Surface 2" terminology** — just call them by name (Salesforce Lead View / Google Sheet).

### SDR surfaces — 8 unified columns (same in both views)

1. Persona (CRO / VP Sales / RevOps / etc.)
2. Company
3. Contact (name)
4. Strongest signal (with evidence)
5. Owner (SDR name OR "Automated outreach")
6. Cadence suggestion (auto-mapped from strongest signal)
7. **Add to sequence** (Yes / No / blank — editable in Sheet only; **NOT a button — it's a column**)
8. Score (combined Intent × Fit)

Salesforce Lead View = read-only mirror. Google Sheet = the editable action surface where SDRs work Monday morning.

### AI hook lifecycle (where it sits, how it's generated)

1. Stage 3 LLM Hook Agent generates 1-2 sentence opener per Tier 1 account proactively during Monday batch
2. Stored in SF custom field `ai_hook`
3. Visible in Sheet (expandable cell or hover-over) for SDR preview
4. Visible on SF account record detail page
5. Pre-fills Amplemarket sequence step 1 body when SDR marks Yes
6. SDR edits/replaces in Amplemarket, composes steps 2-5, sends

---

## Final design — Part 4 (rollout & measurement)

**30-day plan (Alice's exact framing — do not embellish):**
- Week 1: **Talk to the sales team.** How they cut territory, how they run outbound, audit data sources, validate the approach.
- Week 2: **Build & test myself.** Wire all 3 stages, test end-to-end on a 100-account sample.
- Week 3: **First real-time test.** Monday goes live on pilot territory. Daily 15-min standup with SDRs. Iterate.
- Week 4: **System ready.** Onboard remaining SDRs, lock the dashboard.

**5 KPIs (tracked in Looker/Mode → Snowflake + SF):**
1. Scored accounts → meetings booked (overall conversion)
2. Reply rate per cadence + per signal (message resonance)
3. Sequencing rate (% of Tier 1 actioned — SDR adoption)
4. Coverage % (% of accounts firing ≥1 signal weekly)
5. Pipeline → closed-won (90-day cohort)

**4 feedback loops** (NOT 5 — SDR 👍/👎 was dropped):
1. **Cadence-level** — per-cadence reply rate from Amplemarket webhooks → A/B
2. **Agent-level** — LLM hook quality tracked per reply → prompt tuning
3. **Outcome-back-to-source** — AE tags signal driving closed-won (required field on SF Opp at Stage = Closed-Won) → trains Phase 2
4. **Human** — weekly SDR check-in · monthly leadership · quarterly CEO/CRO

### KILLED (do not bring back)

- ❌ **SDR 👍/👎 signal feedback column** — Alice: "it's a dumb system, i don't like it"
- ❌ **Friday Slack report** — Alice dropped it, focus on actionable for SDRs
- ❌ **Real-time velocity alerts** — part of velocity-flag deletion

---

## Slide deck structure (7 slides total)

1. **Cover** — Attention · GTM Engineering Case Study · Alice Doglioli
2. **The Thesis** — 1M → 50k → ~250 actions/week + 4-part outline
3. **Part 1 — 10 signals table** (3 categories, weights, sources, conservative volumes)
4. **Part 2 — Two scores. One 2×2 matrix. Four tiers.**
5. **Part 3a — 1 orchestrator + 3 Python stages + 1 LLM hook agent.** Subtitle: "Scheduled cron · runs every Monday at 6 AM · end-to-end in under 30 minutes."
6. **Part 3b — What the SDR sees Monday morning.** Unified 8-column table mock + Lead View card + Google Sheet card + cadence library card.
7. **Part 4 — 30-day plan · KPIs · feedback · Phase 2.**

**Slide 8 was deleted** — Alice wants closing to be verbal, not deck.

**Eyebrows (purple "Part 1 · Buying Signals" text under headlines) were removed** from slides 3-7 because they duplicated the meta header at top. Slide 2's eyebrow ("The Thesis") is unique and stays.

---

## User preferences (LEARNED — IMPORTANT for future Claude)

- **No AI fluff.** Hates filler subheads, salesy framing, em-dashes connecting clauses. Prefers full words ("with" not "w/", "and" not arrows, periods not em-dashes). Will say things like "I think it's AI fluff bullshit" when frustrated.
- **Honest engineering over hype.** Explicitly endorsed the "2 actual agents, 3 Python stages — no agent hype on what's really ETL" framing. Wants me to push back if her ideas are over-engineered.
- **Defensible decisions.** Wants every choice justified by research or data. Will challenge fabricated numbers (e.g., she caught the made-up "5 SDRs × 50/week = 250").
- **Minimal colors / minimal complexity.** Pushed back on 7-color architecture diagram — wants 3 max.
- **Direct communication.** Will say "no you are dumb" when frustrated. Don't take personally; engage with the substance.
- **Wants pushback when warranted.** Asked me explicitly "give me your critical view, don't just agree" on the 3-agent question. Endorsed honest pushback.
- **Confidentiality:** Won't reference any prior knowledge of Attention's internal tooling on the deck — presents the architecture as her own design.
- **Iterates slide-by-slide.** Don't try to make all changes at once if she's working through one slide.
- **Loves simple, owner-driven mental models** over complex routing (e.g., chose owner-driven cadences over signal-type-driven).
- **Cascades changes for consistency** — if a decision changes Part 1, she expects it cascaded to Part 2/3/4 + slides + system-architecture.html automatically.

---

## OUTSTANDING WORK (pending Alice's confirmation as of last turn)

The session ended with Alice asking deep questions about Part 3 / `system-architecture.html`. I answered her questions verbally and proposed the following changes — **awaiting her confirmation before executing:**

1. **Drop volume callouts** ("~250/week" + "~2,000/week") from architecture end-state nodes
2. **Simplify color palette to 3 categories**:
   - Purple = Agent (orchestrator + LLM hook)
   - Green = Python stages + queries
   - Gray/neutral = sources, storage, outputs
   (Was 7 colors; she said "you have a lot of colors")
3. **Reorganize architecture flow** to emphasize Google Sheet as the primary SDR action surface (not Lead View as parallel output)
4. **Drop Friday Slack report** node + tech-stack mention (she said skip outcomes for now)
5. **Replace "Exclusion Filter (Python)" node** with "Salesforce query" — since SF natively does most of the filtering (territory + customer + opp + churn + layoff state). Only "already engaged in Amplemarket" requires Amplemarket sync, which most teams already have flowing into SF.
6. **Tier 2 = auto-fires by default** (Option A from my Q&A) — SDR doesn't need to mark Yes on Tier 2 rows; they auto-push to Amplemarket programmatic the moment Stage 3 runs. Saves SDR time. SDR can still override via SF if needed.
7. **Make the SDR Yes/No workflow more visually present** in the diagram (currently buried in agent card descriptions)

Alice's last message ended with "Yeah, I think for now let's stop there." — meaning she was pausing for the session, not approving execution.

**When she returns:** ask her to confirm which subset of #1–#7 to execute, then cascade the changes to `system-architecture.html` (and check `part-3.md` for any stale references that drift).

---

## Other key decisions cataloged

- **Schedule mentioned in 3+ places** for visibility: slide 5 subtitle, part-3.md dedicated 🗓️ Schedule section, system-architecture.html lede + Mermaid + agent card.
- **Hard exclusion filter detail moved from slide 3 footnote to slide 5 footnote** — context belongs with architecture, not signals.
- **Snowflake's role:** append-only signal store + Phase 2 ML training data substrate. Velocity computation reference removed.
- **Why we need Snowflake at all:** Salesforce custom objects can't handle 100k+ append-only signal rows at scale; Snowflake is built for this. Already in Attention's stack.
- **Why we write back to SF:** SF is the source of truth for SDR-facing UI (Lead View, account record detail). Scores need to live there for the SDR to see them in their CRM.
- **Goldcast** (signal #2 source) is a B2B virtual events platform — handles webinars + virtual events + attendee tracking. Alice asked what it was; the alternative is HubSpot or Zoom Webinar if Attention doesn't use Goldcast.
- **LinkedIn Sales Nav** (signal #7 source) is the practical answer for buyer-pain-content monitoring. Honest caveat: LinkedIn's public API is restrictive; tools like Common Room or Clay layer on top of Sales Nav for scale. Alice was OK with this.

---

## Research-backed claims (citations available in research-notes.md)

- **Pocus / UnifyGTM:** "First-party signals should always be the highest-weighted inputs in your scoring model" — basis for our weighting.
- **6sense + Bombora:** combined intent sources delivered ~20% lift in pipeline-conversion predictiveness — basis for multi-signal bonus.
- **Apollo / Amplemarket / Clay:** signal-triggered reply rates 8–15% vs. 2–5% baseline cold — north-star metric.
- **Common Room thesis:** "Community engagement precedes purchase by 3-6 weeks" — basis for Slack community signal.
- **Landbase 2026:** Topical engagement (LinkedIn) lifts response rate ~32% — basis for ICP topical engagement.
- **Dreamdata 2026:** "The gap between conversion and engagement is only two days" — basis for ad interaction signal weight.
- **VP Sales tenure ≈ 18 months** (industry benchmark) — basis for executive-change volume estimate.
- **G2 deals run ~2× larger** + competitor layoffs put "every vendor under review" — basis for competitor churn weight.

---

## Quick mental model for future sessions

**The thesis Alice is presenting:** Signal-led GTM with first-party intent leading. 10 weighted signals → 100-point Intent score. Crossed with a Fit score on a 2×2 matrix → 4 tiers. Tier 1 SDRs get an AI-generated hook + compose personally. Tier 2 fires templated cadences automatically. Pipeline runs every Monday at 6 AM. SDRs work in a Google Sheet that mirrors a Salesforce Lead View, both backed by SF as source of truth.

**The framing Alice wants:** honest, defensible, no AI hype, owner-driven simplicity, built on the existing stack, learns from data over time.

**What Alice does NOT want:** velocity flags, signal-type-based cadence routing, AI-drafted full sequences, Surface 1/2 jargon, fabricated capacity numbers, complex color palettes, dumb signal-feedback dropdowns, Friday reports cluttering the architecture.
