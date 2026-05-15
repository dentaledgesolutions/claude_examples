# DentalEdge Solutions — Strategy Brief
**Session date:** May 15, 2026  
**Author:** Erick Sicard (DentalEdge Solutions)  
**Purpose:** Defines the internal growth system to improve proposal quality, competitive positioning, and market intelligence  

---

## What We're Building

An **internal system** (not a client-facing product) that gives DentalEdge three capabilities:

1. **Personalized proposal generation** — automatically creates practice-specific proposals using real data
2. **Case study engine** — captures and presents client results to build social proof
3. **Market intelligence** — monitors dental industry trends to guide service development

These are built in sequence. Phase 1 has direct revenue impact. Phases 2 and 3 compound over time.

---

## The Core Problem This Solves

Current proposal is a polished generic template. Revenue-loss numbers use industry averages, not the prospect's actual numbers. There is no discovery-to-proposal pipeline — data collected verbally never makes it into the document. Prospects don't feel specifically diagnosed.

**Result:** Proposals don't differentiate from competitors when pitching, and some deals are lost because the practice owner doesn't see themselves in the document.

**Target outcome:** A prospect reads the proposal and thinks "they already know my practice."

---

## Phase 1 — Personalized Proposal System

### How it works (end-to-end)

```
Prospect fills Stage 1 form (8 fields, ~90 seconds)
        ↓
GHL workflow triggers
        ↓
Data4SEO API call → pulls Google rating, review count, 
                    local ranking, website health for that practice
        ↓
Claude API call → receives:
  - Competitive intelligence document (all 12 competitors)
  - Stage 1 intake data
  - Data4SEO research results
  → generates personalized proposal content
        ↓
Draft lands in GHL pipeline for Erick's review
        ↓
Approved → sent via GHL Proposals module
           (e-signature + package selection + payment built in)
```

### Stage 1 Intake Form (triggers automation)

| # | Field | Type |
|---|---|---|
| 1 | First Name + Last Name | Text |
| 2 | Email + Phone | Text |
| 3 | Dental Office Name | Text |
| 4 | Role at practice | Dropdown: Owner/Partner · Practice Manager · Associate |
| 5 | Practice website URL | Text (feeds Data4SEO) |
| 6 | City / area | Dropdown: Miami-Dade · Broward · Palm Beach + neighborhood |
| 7 | Specialty | Dropdown: General · Cosmetic · Ortho · Pedo · Multi-specialty |
| 8 | Biggest current challenge | Dropdown: New patients · No-shows · Reviews · Staff overload · All |
| 9 | Current software/tools in use | Multi-select or text (triggers competitor comparison) |
| 10 | Languages spoken at practice | Multi-select: English · Spanish · Creole · Portuguese · Other |

### Stage 2 Form (sent after proposal — prospect is already engaged)

Sent as a follow-up when prospect opens or clicks the proposal. Contains the full operational depth survey currently in GHL:
- Missed calls per day, no-shows per week, overdue recall patients
- Production per visit, marketing budget, AI readiness
- High-value services growth targets
- Front desk time on confirmations
- Lead response time

### Technical Architecture

**Primary path:** GHL-native  
- GHL workflow → HTTP Request step → Data4SEO REST API  
- GHL workflow → HTTP Request step → Claude API (Anthropic)  
- Claude response → GHL Proposals module populates draft  

**Fallback:** If GHL workflow complexity hits a ceiling (JSON parsing between steps), route through Make.com as orchestration middleware. Avoid hosting a custom server unless necessary.

**Access:** Navigate to **Payments → Documents & Contracts** in GHL (no separate activation required — it's a built-in module).

### Competitive Section in Proposal

**Structure:** Full comparison table (all 12 competitors) + a prospect-specific callout paragraph based on field #9 (current tools in use).

**Example:** If prospect selects "Weave" → proposal includes:  
> "You mentioned you're currently on Weave. Weave handles calls. We handle growth. Those aren't the same thing. [continues with specific contrast...]"

Competitive intelligence document (`competitive-intelligence.md`) is the source of truth for all comparison language.

### Review and Send Process

**Initial phase:** Review-then-send. Every draft reviewed and approved by Erick before going to prospect. Switch to instant auto-send after 20–30 approved proposals demonstrate consistent output quality.

---

## Phase 2 — Case Studies and Website Content

### Case Studies (highest-leverage website gap)

**Three active clients to document:**
- Smile Plus
- Companioni Dental Smile
- Palmetto Dental Studio

**Immediate action:** Record baseline metrics for all three today:
- Current Google rating and review count
- Local search ranking (Data4SEO can pull this)
- Key operational metric being addressed (missed calls, no-shows, etc.)

**In 60–90 days:** Use before/after data to build case studies.

**Case study template structure:**
```
Practice: [Name, City, Specialty]
The Problem: [1 paragraph — their situation before DentalEdge, in their words]
What We Did: [2-3 bullets — specific services implemented]
The Results: [3 metrics with before/after numbers]
In Their Words: [Direct quote from owner or manager]
```

**Bridge strategy (while real results build):** Publish anonymized "Scenario Studies" using industry benchmark numbers. Label clearly as "typical results" not client-specific. Replace with real case studies as they become available.

### Website Content Priority Order

1. **Case studies** — social proof closes deals faster than any other content
2. **Local SEO pages** — separate pages for Miami-Dade, Broward, Palm Beach (Google ranks local specificity)
3. **Spanish-language content** — DentalEdge claims multilingual expertise but doesn't demonstrate it on site
4. **Blog / content engine** — SEO traffic growth, lower priority until Phase 3

---

## Phase 3 — Market Intelligence System

### Delivery Format

- **Monthly digest:** Claude reads key sources, generates 1-page summary of what's emerging in dental marketing and practice tech. Delivered to Erick's inbox.
- **Quarterly strategy brief:** Maps top 3 trends to specific DentalEdge service opportunities. Format: "Trend → Implication → Recommended Action → Target Quarter."

### Sources to Monitor

| Source | Focus |
|---|---|
| Dental Economics | Business of dentistry, practice management |
| Dental Products Report | New technology and software launches |
| Group Dentistry Now | DSO moves and competitive pressure |
| The Dental Marketer (newsletter/podcast) | Dental marketing tactics |
| ADA Health Policy Institute | Patient behavior data, workforce trends |
| Florida Dental Association | State-level regulations, CE trends |
| Competitor blogs (Weave, RevenueWell, Dental Intelligence, Adit) | Competitor feature releases |
| Google Trends | "dental AI", "dental marketing" query trends |

### Emerging trends to watch now

- **Generative Engine Optimization (GEO)** — practices need to appear in ChatGPT and Perplexity results, not just Google. DentalEdge can build this as a service before competitors.
- **DSO expansion in SE Florida** — Aspen Dental and Heartland investing heavily in Miami-Dade. Independent practices need sharper differentiation.
- **AI voice agents becoming table stakes** — window to differentiate on AI is closing. Lead with implementation speed and managed service.
- **Multilingual AI** — Spanish-language AI voice agents underserved in SE Florida market. Clear opportunity.

---

## Competitive Positioning Summary

**DentalEdge's defensible position:** The only done-for-you dental growth system in Southeast Florida built on a unified platform (GoHighLevel), priced for independent practices.

**The one-line contrast:** Every competitor hands you software. We hand you results.

**Full competitive intelligence:** See `competitive-intelligence.md` (12 competitors profiled, 1-paragraph objection responses, comparison matrix, 4 common objections scripted).

---

## Build Sequence and Next Steps

### Immediate (this week)
- [ ] Record baseline metrics for Smile Plus, Companioni Dental Smile, and Palmetto Dental Studio
- [ ] Confirm Documents & Contracts access: **Payments → Documents & Contracts** in GHL left menu
- [ ] Add 3 missing fields to existing GHL survey: website URL, current tools/software, specialty

### Phase 1 build (proposal system)
- [ ] Design Stage 1 form in GHL (8–10 fields defined above)
- [ ] Build GHL workflow: form submission → Data4SEO API call → Claude API call → GHL pipeline entry
- [ ] Write Claude system prompt using `competitive-intelligence.md` as context
- [ ] Test with 3 mock prospects before going live
- [ ] Build proposal template in GHL Documents & Contracts (Payments → Documents & Contracts → New Document)
- [ ] Define review/approval process in GHL pipeline stages

### Phase 2 build (case studies)
- [ ] Build case study template (structure defined above)
- [ ] Publish 3 "scenario study" placeholders on website now
- [ ] Replace with real case studies as Smile Plus, Companioni, and Palmetto results mature (60–90 days)
- [ ] Draft local SEO pages for Miami-Dade, Broward, Palm Beach after first real case study published

### Phase 3 build (trend tracking)
- [ ] Subscribe to all 8 sources listed above
- [ ] Build Claude prompt for monthly digest generation
- [ ] Set up GHL or calendar trigger for monthly + quarterly runs
- [ ] First quarterly strategy brief targets Q3 2026

---

## Success Metrics

| Metric | Target | Timeframe |
|---|---|---|
| Proposal personalization | 100% of proposals include prospect's real Google data | Phase 1 launch |
| Proposal review time | Under 15 minutes per proposal | Phase 1 launch |
| Proposal-to-close rate | Measurable baseline established | 30 days post-launch |
| Case studies live | 3 real case studies published | 90 days |
| Competitive objections handled | 0 deals lost to "I don't know how you compare to X" | 60 days |
