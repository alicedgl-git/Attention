# Part 3: System Design & GTM Integration

**The brief asks:** *"What does the end product look like? Where does it live? How do we reliably track signals and score accounts? Which accounts go to SDRs vs. automated outreach? How prescriptive on outreach? How does this fit the SDR's day-in-the-life?"*

**The system, in one paragraph:** A weekly job runs every Monday at 6 AM, supervised by a **Claude Code orchestrator agent** (cron + retry/error-handling). The pipeline starts from a **Salesforce query** — territory + status filters do most of the exclusion work natively (drops customers, active opps, churned, layoffs, already-engaged accounts). The orchestrator then runs **3 Python pipeline stages**: **Stage 1 — Signal Collector (Python ETL)** calls 10+ APIs (Clay, RB2B, GA4, LinkedIn Campaign Manager, Reddit Ads, Common Room, Crunchbase, G2, Amplemarket, Salesforce) to pull raw signals into Snowflake. **Stage 2 — Scorer (Python script)** computes Fit Score + Intent Score + multi-signal bonus, applies the 2×2 routing matrix, writes scores back to Salesforce. **Stage 3 — Distributor (Python + LLM Hook Agent)** is owner-driven: for Tier 1 (SDR-owned) accounts, the LLM Hook Agent generates a 1–2 sentence personalized opener stored on the SF account; for Tier 2 (Auto-owned) accounts, Python **auto-pushes** to a templated Amplemarket sequence with custom-field tokens (no LLM, no SDR action). SDRs work in a per-territory Google Sheet (editable "Add to sequence" column) that mirrors a native Salesforce Lead View. Tier 1 rows wait for SDR Yes → AI hook pre-fills Amplemarket step 1 + SDR composes the rest. Tier 2 rows show as "Sequenced" by Monday 6:31 AM with no SDR action required.

**Why this framing matters:** "Agents" is over-applied in modern GTM-tooling marketing. The honest split: **2 actual agents** (orchestrator + hook generator) and **3 Python pipeline stages**. The LLM is invoked at exactly 2 specific points — sub-type classification (Stage 1) and hook generation (Stage 3). Everything else is deterministic ETL, where Python beats agents on cost, reliability, and debuggability.

---

## 🗓️ Schedule

**The system runs as a scheduled weekly cron job — every Monday at 6 AM.** End-to-end runtime: under 30 minutes. By the time SDRs open their Sheet at 8 AM Monday, everything is ready: signals collected, accounts scored, tiers assigned, AI hooks generated, Tier 2 accounts pushed to Amplemarket, Sheets refreshed, SF Lead View updated.

The schedule is locked at the orchestrator level (Claude Code cron). No on-demand runs in v1 — that's a Phase 2 enhancement (real-time webhook-driven re-scoring on rare-but-hot signals like Competitor churn).

---

## The pipeline in 3 stages

```
[Salesforce — 50k ICP universe]
            ↓
[Salesforce query · territory + status filters]
   → drop ~5–8%: customers, active opps, churned <6mo,
     layoffs at prospect, already-engaged in Amplemarket <90d
            ↓
[~40k available accounts]
            ↓
┌─────────────────────────────────────────────────────┐
│  CLAUDE CODE ORCHESTRATOR (the agent)               │
│  Cron · retry/error-handling · cross-stage supervision │
└─────────────────────────────────────────────────────┘
            ↓ runs the 3 stages in sequence
┌─────────────────────────────────────────────────────┐
│  STAGE 1 — Signal Collector (Python ETL)            │
│  • Fan out 10+ API calls in parallel                │
│  • [LLM micro-call] sentiment classifier on #7      │
│  • [LLM micro-call] role classifier on #5           │
│  → writes raw signals (append-only) to Snowflake    │
└─────────────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────────────┐
│  STAGE 2 — Scorer (Python script, deterministic)    │
│  Reads Snowflake + SF metadata                      │
│  Computes: Fit · Intent · Multi-signal bonus →      │
│            2×2 matrix → Tier 1/2/3/4                │
│  → writes scores back to Salesforce                 │
└─────────────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────────────┐
│  STAGE 3 — Distributor (Python + LLM hook agent)    │
│  For Tier 1 [LLM AGENT]: generates AI hook from     │
│             cadence playbook + signal evidence      │
│  For Tier 2 (Python): pushes templated cadence to   │
│             Amplemarket via API                     │
│  Refresh: Google Sheets per territory (Apps Script) │
│  SF Lead View auto-updates from Stage 2 SF writes   │
└─────────────────────────────────────────────────────┘
       ↓                        ↓
[SF Lead View +              [Amplemarket
 Google Sheet]                programmatic]
       ↓                        ↓
   SDRs (Tier 1)            Auto-outbound (Tier 2)
       ↓                        ↓
       └──── Outcomes → Salesforce ────┘
                    ↓
       [Outcomes captured for Phase 2 calibration]
```

**Honest agent count:** 2 actual agents (the **Claude Code orchestrator** + the **LLM hook agent** in Stage 3). 3 Python pipeline stages. 2 inline LLM classification calls in Stage 1. Everything else is deterministic Python — cheaper, more reliable, easier to debug than agentizing pure ETL.

---

## Stage 1 — Signal Collector (Python ETL)

**Type:** Python ETL pipeline, *not* an agent. Cron-triggered.
**Trigger:** Monday 6 AM after the Salesforce query returns the available account list.
**Input:** ~40k available account IDs from Salesforce.
**Output:** raw signal records in Snowflake — `(account_id, signal_id, evidence, fired_at, source, raw_payload)`. Append-only, never overwrites.
**LLM work (inline, not full agents):** small classification calls on signals #5 (senior exec vs. operational hire) and #7 (pain-coded vs. neutral topical post). Each is a single per-record LLM API call (~100ms, ~$0.0001), parallelizable, retryable. Treated as a tool inside the Python pipeline, not a separate agent.

### API / tool mapping per signal

| # | Signal | API / Tool | What we extract |
|---|---|---|---|
| 1 | Website visits | RB2B (deanon) + GA4 (page events) | Company-level visits to /demo, /pricing, /product, /integrations, /customers, /solution — with persona match |
| 2 | Content engagement | HubSpot / Marketo (marketing automation) + Goldcast / Demio (webinars) + LinkedIn API (page engagement) + Spotify/Apple podcast tracking | Webinar registrants/attendees, content downloads, podcast listens, social engagement |
| 3 | Ad interaction | LinkedIn Campaign Manager API + Reddit Ads Manager API | Click-throughs, video views ≥25%, comments, saves/shares, lead-gen form opens |
| 4 | Slack community | Common Room API (Slack/Discord/Discourse capture) + Champify-style alerts | New ICP-role members + topical posts in target communities |
| 5 | Executive change / Key role posts | Clay API (LinkedIn Sales Nav job-change tracking) + LinkedIn Jobs + SF contact title-change history + TechCrunch / PR Newswire APIs | New hires/promotions in VP Sales / CRO / Head of / RevOps / Enablement / Sales Ops in last 90 days; open postings for these roles |
| 6 | Sales team scaling | Clay API (LinkedIn Jobs) + Greenhouse / Lever ATS scrapers | Count of AE/SDR/CSM/AM postings per company in last 30 days |
| 7 | Buyer pain / ICP topical | Clay API (LinkedIn social monitoring) + Common Room + Listen Notes API (podcasts) + Quartr / AlphaSense (earnings transcripts) | ICP-role posts + sentiment classifier (pain vs. neutral) |
| 8 | Competitor usage | Clay API (LinkedIn Jobs) + Greenhouse / Lever | JD keyword scan for "Gong / Chorus / Clari / Outreach / Salesloft experience required" |
| 9 | Competitor churn window | G2 Buyer Intent API (review-rating delta on competitor pages) + layoffs.fyi + LinkedIn employee-count tracking on competitor pages + Crunchbase News + win/loss intel (custom SF field) | G2 review drops, competitor layoffs/departures, known contract-end windows |
| 10 | Recent funding | Crunchbase API + PitchBook (via Clay) + TechCrunch / Axios Pro Rata + SEC EDGAR | Series A+ funding events in last 6 months |
| — | **Exclusion inputs** (pre-filter) | Amplemarket API + Salesforce | Engagement state (replied/clicked/booked <90d), opp/customer/churn state |

**Parallelism:** the agent fans out the 10 API calls per account in parallel; full 40k-account run completes in under 30 minutes.

---

## Stage 2 — Scorer (Python script)

**Type:** pure deterministic math. No LLM, no agent — straight Python.
**Trigger:** when Stage 1 finishes writing to Snowflake.
**Input:** raw signals from Snowflake + account metadata from SF (industry, employee count, sales rep count, funding stage).
**Output:** scoring records in SF — `{account_id, fit_score, intent_score, tier, signals_fired[], strongest_signal}`.

**What it computes:**

1. **Fit Score** (0–100, monthly cache, refreshed if stale): Industry tier 50 + Sales motion intensity 30 + Growth/funding stage 20.
2. **Intent Score** (0–100+, weekly): `Σ (signal_weight if fired) + multi_signal_bonus`.
3. **Tier assignment** via 2×2 matrix:
   - High Fit (≥70) × High Intent (≥30) → **Tier 1 — SDR**
   - Low Fit × High Intent → **Tier 2 — Auto-outbound**
   - High Fit × Low Intent → **Tier 3 — Watchlist**
   - Low Fit × Low Intent → **Tier 4 — Skip**
5. **Strongest signal** picked by weight, with sub-type from Stage 1's classification (used by Stage 3 for cadence routing).

Pure deterministic Python. No LLM judgment needed — every input maps to one output. The orchestrator (Claude Code) catches edge cases (missing fields, account merges, fit-score-stale recompute decisions) at the supervision layer; Stage 2 itself stays simple.

---

## Stage 3 — Distributor (Python + LLM hook agent)

**Type:** hybrid. The AI hook generator is a real LLM agent. The rest is Python distribution code.
**Trigger:** when Stage 2 finishes scoring.
**Input:** scored accounts with tier assignments + strongest signal + signal evidence.
**Output:** SF Lead View populated, Google Sheets refreshed, Amplemarket campaigns triggered.

**What it does:**

**Routing is owner-driven, not signal-type-driven.**

1. **For Tier 1 accounts (SDR-owned) [LLM Hook Agent]:**
   - LLM Hook Agent generates a 1–2 sentence opener using signal evidence + buyer profile (proactively, during Monday batch — ~250 calls/week).
   - Validated (length + brand-safety) with retry + templated fallback if QC fails.
   - Written to SF custom field `ai_hook` on the account record.
   - When SDR flips "Add to sequence" to Yes, Apps Script creates a draft Amplemarket sequence: AI hook pre-filled in step 1 body, 5-touch / 14-day templated structure for steps 2–5 (all editable).
   - SDR opens Amplemarket, composes the email in their own voice using hook + signal context as research, sends.

2. **For Tier 2 accounts (Auto-outreach) [Python only — no LLM]:**
   - Python populates SF custom-field tokens for the matching templated cadence based on signal evidence (e.g., `webinar_name = "AI call coaching at scale"`, `competitor_name = "Gong"`).
   - Writes `recommended_cadence` field back to Salesforce.
   - When SDR flips "Add to sequence" to Yes, Apps Script pushes contact (with custom fields populated) to the matching generic Amplemarket sequence → Amplemarket renders tokens at send time → fires automatically.
   - SDR can override the cadence suggestion by editing the cadence cell before marking Yes.

3. **Refresh Google Sheets [Python]** (one per SDR territory) via Apps Script — pulls from SF, formats the 8 unified columns. Apps Script listens on the "Add to sequence" column edit event and routes by **owner** (SDR vs Automated) per the logic above.

4. **Salesforce Lead View** auto-updates from Stage 2's SF write-back (no separate write needed). AI hook visible on the account record detail page.

**Why LLM is bounded to hook generation only:** AI is genuinely good at generating a single 1–2 sentence opener from structured signal evidence. AI is less good at writing full bespoke sequences that match SDR craft on high-stakes accounts. Owner-driven routing puts AI where it adds value (research + opener prep) and humans where they add value (craft + judgment on high-stakes pitches). LLM cost is bounded to ~250 short calls/week — trivial.

---

## The end product — Salesforce Lead View + Google Sheet + auto-outbound

Both views fed by the same Salesforce write-back. **Same 8 columns** in both — two views into one source of truth, no duplicate work.

> ## ⚠️ CRUCIAL DESIGN DECISION (read carefully)
>
> Earlier iterations had Lead View and Google Sheet showing **different column sets** (Fit / Intent / Last interaction / Status / SDR feedback dropdown / etc.) and used a per-row **"Add to sequence" *button*** in the Sheet to trigger Amplemarket.
>
> **The final design unifies both views around ONE 8-column table** (see below), and replaces the per-row button with an **editable "Add to sequence" *column*** (Yes / No / blank).
>
> **Why this matters:**
> - Bulk-action friendly: SDR scans the whole list, marks Yes/No on multiple rows, processes in one batch — better Monday-morning workflow than clicking 50 buttons
> - Structurally identical views = zero confusion when SDR moves between SF and Sheet — they see the same shape
> - Status semantics are explicit: **Yes** = sequenced this week · **No** = explicitly skipped this week · **blank** = not yet decided
> - Apps Script trigger is on the *column edit event* (when SDR flips a cell to Yes), which fans out: LLM hook generator (Stage 3 LLM agent) → Amplemarket API push → SF write-back of "sequenced" status
> - "Surface 1 / Surface 2" terminology is dropped — the views are just called by name (Salesforce Lead View / Google Sheet). No umbrella term needed.

### The unified 8-column table

| # | Column | Notes |
|---|---|---|
| 1 | **Persona** | Role of the contact: VP Sales / CRO / RevOps Lead / SDR Manager / Sales Enablement / etc. |
| 2 | **Company** | Account name + SF link |
| 3 | **Contact** | Best contact name + LinkedIn / email |
| 4 | **Strongest signal** | The highest-weight signal that fired this week (per Part 1's ranking — Website visits 20 outranks Content engagement 14 outranks Exec change 12, etc.). Includes signal name + brief evidence (e.g., "Website visits · /demo + /pricing" or "Competitor churn · G2 drop on Gong"). |
| 5 | **Owner** | SDR name for Tier 1 accounts (high Fit + high Intent). "Automated outreach" for Tier 2 accounts (low Fit + high Intent). Determined by the routing matrix in Part 2. |
| 6 | **Cadence suggestion** | **Owner-driven.** Tier 1 (SDR-owned) → "Personalized — AI hook ready" (the AI-generated hook itself sits in SF `ai_hook` and is visible via expandable cell; pre-fills Amplemarket step 1 when SDR marks Yes). Tier 2 (Auto-owned) → matched templated cadence name from the 10-cadence library; SDR can override before marking Yes. See Cadence playbook below. |
| 7 | **Add to sequence** | **Editable in Google Sheet only (Yes / No / blank).** Yes → Apps Script triggers LLM hook + pushes contact + cadence to Amplemarket via API. In Lead View this column is read-only and reflects the Sheet's state. |
| 8 | **Score** | Combined `Intent × Fit` shorthand — single number for quick prioritization. Backs to the matrix in Part 2 (e.g., 89 = High Fit × High Intent → Tier 1). |

### Salesforce Lead View

Native SF list view, filtered per-rep. Lives inside the SDR's existing Salesforce home — zero adoption friction, no new tool to learn.

- Filter: `tier IN ("Tier 1", "Tier 2") AND owner = current_user AND status != "Disqualified"`
- Sort: Score DESC, then Strongest signal recency
- All 8 columns visible · all read-only (the SDR triages here, but actions happen in the Sheet)
- Used for: in-CRM triage, account-record context, mass-update non-action fields

### Google Sheet

Per-territory, refreshed Mon 6:30 AM. Same 8 columns as Lead View, with the **Add to sequence** column editable.

- SDR scans the list, marks **Yes / No / blank** per row
- "Yes" rows are routed by **owner**:
  - **Tier 1 (SDR-owned):** Apps Script creates a draft Amplemarket sequence with the AI hook (already on SF from Stage 3) pre-filled in step 1 + a 5-touch templated structure for steps 2–5. SDR opens Amplemarket, composes their email, sends. Status → "Sequenced."
  - **Tier 2 (Auto-owned):** Apps Script pushes contact to the matching templated Amplemarket sequence with custom-field tokens populated by Stage 3. Fires automatically. Status → "Sequenced."
- "No" rows → explicitly skipped this week. Re-scored next Monday (could become Yes next week if signals change).
- Bulk-action friendly: SDR can mark multiple rows in one pass.

Optional secondary tabs (kept context-only, no actions):
- **Tab 2: Watchlist** — Tier 3 accounts (High Fit, Low Intent)
- **Tab 3: Recently sequenced** — last 60 days, status updates from Amplemarket

### Auto-outbound (Tier 2)

Amplemarket programmatic, no SDR involvement. Owner = "Automated outreach" in the table. ~2,000 accounts/week pushed automatically every Monday after Stage 3 runs. Replies routed to a shared inbox monitored by the growth team. No AI custom hook — templated openers per cadence at zero per-touch cost.

---

## Cadence playbook — owner-driven workflow

> ## ⚠️ DESIGN DECISION (locked)
> Earlier iterations split the cadence library by **signal type** (templated vs AI-assisted). Final design splits by **owner** (SDR vs Automated outreach). Reason: AI is bounded to the one job it's actually good at (generating hooks); SDRs own the craft on high-stakes accounts where AI-drafted full sequences would underperform a human-written email. Cleaner, cheaper, more honest about where AI adds value.

The owner determines the workflow. The signal type only determines the *cadence shape* (which template the Auto path uses).

### Tier 1 — SDR-owned: Personalized + AI hook

**Cadence column shows:** "Personalized — AI hook ready"
**Workflow:**
1. Stage 3 LLM Hook Agent has already generated a 1–2 sentence opener for the account (proactive, during Monday batch). Stored in SF custom field `ai_hook`. Visible in the Sheet as a previewable cell.
2. SDR reads the hook + signal context in the Sheet, decides if the account is worth pursuing.
3. SDR marks **Yes** in Add to sequence column.
4. Apps Script creates a draft sequence in Amplemarket: AI hook pre-filled in step 1's body, 5-touch / 14-day templated structure for steps 2–5 (sales leadership defines the shape, body editable).
5. SDR opens Amplemarket, edits step 1 (often replaces hook with their own version after using it as a starting point), writes steps 2–5 in their own voice, sends.

**Time:** ~10–15 min per account. SDR is doing the craft; AI does the prep.

**Why this works:** the strongest accounts (CRO joining 45 days ago / VP Sales posting about forecast pain / Series B'd company evaluating us on pricing page) deserve a human-written email. AI gives the SDR a starting point + research; the SDR brings craft, judgment, and brand voice. Reply rates on SDR-written hooks should beat AI-drafted ones on bespoke signals.

### Tier 2 — Auto-outbound: templated cadence library

**Cadence column shows:** matched cadence name from the library (e.g., "Webinar / event followup")
**Workflow:**
1. Stage 3 Python populates SF custom-field tokens for the matching templated cadence based on signal evidence (e.g., `webinar_name = "AI call coaching at scale"`, `pages_visited = "/demo, /pricing"`, `competitor_name = "Gong"`).
2. SDR (optionally) reviews the cadence suggestion. Can override by editing the cadence cell to point to a different cadence in the library.
3. SDR marks **Yes** (or **No** to skip).
4. Apps Script pushes the contact (with custom fields populated) to the matching generic Amplemarket sequence → Amplemarket renders tokens at send time → fires automatically. No LLM, no review step.

**Time:** ~30 sec per account (just Yes/No, optionally reroute).

### The 10 templated cadences (Auto-outreach only)

One generic Amplemarket sequence per signal, owned by sales leadership. Each uses custom-field tokens for the signal-specific data:

| Signal | Cadence name | Sample custom-field tokens |
|---|---|---|
| #1 Website visits | Website visits | `{{pages_visited}}` · `{{visit_count}}` |
| #2 Content engagement | Webinar / event followup | `{{event_name}}` · `{{event_date}}` · `{{event_topic}}` |
| #3 Ad interaction | Ad interaction followup | `{{ad_topic}}` · `{{platform}}` |
| #4 Slack community | Slack community | `{{community_name}}` · `{{post_topic}}` |
| #5 Executive change | Sales leadership change | `{{new_exec_name}}` · `{{new_exec_title}}` · `{{previous_company}}` |
| #6 Sales team scaling | Sales team scaling | `{{role_count}}` · `{{role_types}}` |
| #7 Buyer pain / topical | Buyer pain / topical | `{{post_topic}}` · `{{post_excerpt}}` |
| #8 Competitor usage | Competitor usage | `{{competitor_name}}` · `{{competitor_role_count}}` |
| #9 Competitor churn | Competitor churn | `{{competitor_name}}` · `{{churn_event_type}}` |
| #10 Recent funding | Recent funding | `{{round_size}}` · `{{round_stage}}` · `{{lead_investor}}` |

**No per-contact LLM** in this path. Sales leadership owns 100% of body copy. Growth eng owns the Python that populates tokens.

### How the workflow branches

```
SDR opens Sheet, scans accounts
         ↓
For each row, marks Yes / No / blank
         ↓
Apps Script reads OWNER column
         ↓
  ┌──────┴──────┐
  ↓             ↓
SDR (Tier 1) AUTOMATED (Tier 2)
  ↓             ↓
Apps Script   Apps Script
creates DRAFT pushes contact
sequence in   to matching
Amplemarket:  templated Amplemarket
  · AI hook   sequence (custom
    in step 1 fields populated
  · Templated by Stage 3)
    structure   ↓
    steps 2–5 Amplemarket
  ↓             renders tokens
SDR opens     → fires automatically
Amplemarket,  
edits + writes
own copy → sends
```

### Cadence ownership

- **Templated cadences (Auto-outreach):** Sales leadership owns 100% of body copy in Amplemarket. Growth eng owns the Python that populates SF custom-field tokens per contact based on signal evidence. No LLM in this path.
- **Personalized (SDR):** Sales leadership defines the cadence shape (5–6 touches over 14–30 days, brand-voice guidelines). Growth eng owns the LLM Hook Agent (Stage 3). SDR owns the email composition.

### AI hook lifecycle (where it sits, how it's generated)

| Stage | What happens |
|---|---|
| **1. Generation** | Stage 3 LLM Hook Agent runs for every Tier 1 account during the Monday batch (~250 calls/week, ~$0.001 each). Inputs: strongest signal + evidence + buyer profile (name, title, company). Output: 1–2 sentence opener. Validated for length + brand-safety; fallback prompt if QC fails. |
| **2. Storage** | Written to SF custom field `ai_hook` on the account record. SF is source of truth. |
| **3. Preview in Sheet** | Cadence column shows "Personalized — AI hook ready"; the hook itself accessible via expandable cell or hover-over. SDR can read it before deciding Yes/No. |
| **4. Preview in Lead View** | Visible on the SF account record detail page (standard SF UX). |
| **5. Pre-fill in Amplemarket** | When SDR marks Yes, Apps Script creates a draft sequence: AI hook pre-filled in step 1 body, 5-touch / 14-day templated structure for steps 2–5 (all editable). |
| **6. SDR composition** | SDR opens Amplemarket, edits step 1 (replaces or refines hook), writes steps 2–5, sends. |

**Hook is proactively pre-generated** (during Monday batch) so it's ready when SDR opens the Sheet at 8 AM. No on-demand waiting. Cost: ~$13/year at this volume — trivial.

**Concurrent dependency:** the templated cadence library doesn't yet exist at Attention — building it (10 generic Amplemarket sequences with custom-field tokens, deliverability rules, brand voice) is a parallel workstream owned by sales leadership + growth eng. Worth flagging in the interview.

---

## SDR day-in-the-life

| When | What happens |
|---|---|
| **Mon 6:00 AM** | Cron fires. Agents 1 → 2 → 3 run end-to-end (~30 min). |
| **Mon 6:30 AM** | SF Lead View updated. Google Sheets refreshed. Amplemarket Tier 2 campaigns live. |
| **Mon 8:00 AM** | SDR opens Lead View for triage, then Sheet for action. Scans the 8-column table. <strong>For Tier 1 (SDR-owned) rows:</strong> reads the AI hook (expandable cell), marks Yes if worth pursuing → Apps Script creates Amplemarket draft (hook in step 1) → SDR opens Amplemarket, composes the email in their voice, sends. ~10–15 min/account. <strong>For Tier 2 (Auto-owned) rows:</strong> reviews the suggested cadence (optionally reroutes), marks Yes → Apps Script pushes to the matching templated sequence; fires automatically. ~30 sec/account. ~50 Tier 1 + ~80 Tier 2 per SDR per week. |
| **Tue–Thu** | Work residual Tier 1 + handle replies coming back through Amplemarket → SF. |
| **Fri 4:00 PM** | Slack report (per SDR + #growth): "X Tier 1 sequenced, Y replies, Z meetings booked. Top-converting signal: [signal]." SDR fills feedback column on signals that were unusually good or useless. |
| **Continuous** | Auto-outbound (Tier 2) runs unattended via Amplemarket. Replies → growth team shared inbox → AE handoff for booked meetings. |

---

## Tech stack mapping

| Tool | Role |
|---|---|
| **Salesforce** | Source of truth. 50k account universe, ICP filter, territory ownership, score/tier write-back, opp/customer status, outcome tracking. Hosts the per-rep Lead View. |
| **Clay** | Enrichment + LinkedIn / ATS / Crunchbase / news orchestration. Powers most of Agent 1's signal collection. |
| **Snowflake** | Warehouse. Append-only signal store + scoring history + Phase-2 ML training data. |
| **Amplemarket** | Outbound execution. SDR sequences (Tier 1) + programmatic auto-outbound (Tier 2). API-driven push from Agent 3. Engagement state feeds the exclusion filter. |
| **Attention** | (eventually) Conversation intelligence on resulting calls — closes the loop. Outcomes feed Phase-2 model training. |
| **Claude Code** | Orchestrator agent + supervisor. Cron + retry/error handling across the 3 Python pipeline stages. The actual LLM is invoked at 2 specific points: sub-type classification (Stage 1) and AI hook generation (Stage 3). |
| **Google Sheets** | SDR-facing UI. Per-territory cockpit. Apps Script handles "Add to sequence" → Amplemarket API. |
| **Slack** | (optional, Phase 2) Real-time alerts on rare-but-hot signals. Not in v1. |
| **External feeds** | RB2B / Clearbit Reveal / 6sense / G2 / LinkedIn Campaign Manager / Reddit Ads / Common Room / Crunchbase / PitchBook / Listen Notes / Quartr — all called by Agent 1, results landed in Snowflake. |

No new tools required. Everything bolts onto the existing stack.

---

## Phase 2

Replace heuristic Intent weights with a logistic regression trained on Attention's closed-won/lost data. Layer a per-industry weight tuner once 90 days of pilot data is available. Add an AE-feedback loop where closed deals retroactively tag which signal drove conversion.
