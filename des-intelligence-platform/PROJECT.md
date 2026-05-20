# DES Intelligence Platform — Project Plan

**Owner:** DentalEdge Solutions  
**Market:** SE Florida — Miami-Dade, Broward, Palm Beach  
**Primary Goal:** Give DES a research engine + proposal machine that generates personalized, data-backed Practice Proposals in minutes and keeps DES ahead of market trends.

---

## What This Platform Does

Two layers working together:

**Layer 1 — Knowledge Base** (built once, refreshed periodically)
A living collection of research documents: who DES is, what the SE Florida dental market looks like, how DES compares to competitors, and what trends are emerging. Every proposal draws from this layer.

**Layer 2 — Operational Workflow** (runs per prospect)
A triggered pipeline: prospect fills out a GHL audit form → Claude pulls the data via GHL MCP → Claude enriches it with Data4SEO → Claude generates a personalized HTML proposal → DES reviews and pushes to GHL when ready.

The platform is built using GSD phases and exposed through custom Claude Code skills.

---

## Methodology

**Research-first, proposal-engine-first.** The Knowledge Base (Layer 1) and the technical infrastructure (Layer 2) are built in parallel. The first operational milestone is a working `/des-proposal [contact-id]` command. The Intelligence Layer (trends monitoring, content pipeline) is Milestone 2.

Each phase is a GSD phase: planned, executed, and verified before the next begins. No phase ships without its skill working end-to-end.

---

## Technology Stack

| Tool | Role |
|------|------|
| Claude Code | Orchestration, research, generation, skills runtime |
| GHL MCP | Pull audit form data, push proposals to contacts, publish blog content |
| Data4SEO | Prospect data enrichment (local SEO rankings, competitor comparison, GBP signals) |
| markitdown | Parse `.docx`, `.html`, `.pdf` source documents |
| Reddit (public API) | Trends monitoring — social signals |
| LinkedIn (web fetch) | Trends monitoring — professional signals |
| YouTube (web fetch) | Trends monitoring — industry content signals |
| context-mode MCP | Knowledge base indexing and search |
| claude-mem MCP | Persistent memory across sessions |
| uv / Python 3.13 | Runtime for markitdown and data processing scripts |

---

## Project Structure

```
des-intelligence-platform/
├── CONTEXT.md                         ← Domain glossary (this is authoritative)
├── PROJECT.md                         ← This file
├── .claude/
│   └── settings.local.json
├── .planning/
│   ├── ROADMAP.md
│   └── docs/adr/                      ← Architecture decisions
├── knowledge/
│   ├── company-profile.md             ← Company Study output — DES ground truth
│   ├── market-study.md                ← Market Study output — SE Florida landscape
│   ├── benchmarking.md                ← DES vs competitors
│   ├── services.md                    ← Service Portfolio — single source of truth
│   └── intelligence/
│       ├── trends.md                  ← Monthly trends document (auto-updated)
│       ├── sources.md                 ← Tracked publications, sites, channels
│       └── briefs/                    ← Content opportunity briefs
├── audit/
│   ├── form-design.md                 ← Audit form questions and rationale
│   └── submissions/                   ← Processed audit data per practice
├── proposals/
│   └── [practice-name]-[date].html    ← Generated proposals
├── content/
│   └── published/                     ← Record of GHL-published content
└── .claude/skills/
    ├── des-proposal/                  ← /des-proposal skill
    ├── des-audit/                     ← /des-audit skill
    ├── des-trends/                    ← /des-trends skill
    └── des-content/                   ← /des-content skill
```

---

## Milestone 1: Foundation & Proposal Engine

### Phase 1 — Project Foundation
**Goal:** Establish the full project structure and verify all integrations work.

**Tasks:**
- Create directory structure above
- Seed `knowledge/services.md` from existing proposal (`DentalEdgeSolutions_Proposal.html`) — packages, à la carte services, pricing, competitive table
- Verify GHL MCP connection (pull a test contact)
- Verify Data4SEO credentials (run a test domain lookup)
- Seed `knowledge/intelligence/sources.md` with initial tracked sources (ADA News, Dental Economics, Dentistry Today, key Reddit subs, competitor domains)
- Initialize `.planning/ROADMAP.md` with all phases

**Deliverable:** Verified integrations, populated `services.md`, initialized knowledge structure.

---

### Phase 2 — Company Study
**Goal:** Capture DES's complete business profile as a structured reference document.

**Method:** Structured interview conducted in Claude Code. Claude asks, Erick answers, Claude writes.

**Interview covers:**
- Services offered and their exact scope
- Pricing (packages + à la carte) and any unpublished nuances
- Target client profile — ideal practice type, size, owner persona
- Key differentiators vs. competitors
- Problems DES focuses on solving
- Geographic boundaries and any planned expansion
- Onboarding process and implementation timeline
- HIPAA compliance approach
- Current client base and any testimonials

**Deliverable:** `knowledge/company-profile.md` — the authoritative DES reference. Every proposal, every content piece, and every market comparison reads from this document.

---

### Phase 3 — Market Study
**Goal:** Build a research-backed picture of the SE Florida independent dental practice market.

**Research areas:**
1. Top 7–10 problems facing independent dental practices in Miami-Dade, Broward, Palm Beach
2. DSO competitive pressure in SE Florida — which DSOs are most active, what they offer
3. Direct competitors to DES: Weave, RevenueWell, Adit, MaxAssist, Aron AI, DoctorLogic — services, pricing, SE Florida presence
4. Tools most commonly used by SE Florida practices (PMS, communication, marketing)
5. Multilingual patient demographics and their service implications
6. Local search dynamics — how practices get found in SE Florida vs. other markets

**Data sources:** Data4SEO (keyword research, local rankings), web fetch (competitor sites), markitdown (industry reports), Reddit/LinkedIn

**Deliverable:** `knowledge/market-study.md` — the market reference. Directly informs audit form design in Phase 4.

---

### Phase 4 — Practice Audit System
**Goal:** Build the audit form and the `/des-audit` skill that powers proposal generation.

**Audit form design:**
- Questions derived directly from Market Study top problems
- Organized into sections: Practice Profile, Digital Presence, Patient Flow, Operations, Revenue, Goals
- Created in GHL (form builder) and documented in `audit/form-design.md`
- Each question maps to one or more DES services that address the problem

**`/des-audit [contact-id]` skill:**
- Pulls GHL contact record and audit form submission via GHL MCP
- Calls Data4SEO for the practice's domain: local keyword rankings, competitor comparison (top 3 nearby practices), GBP signals
- Produces a structured `AuditResult` document saved to `audit/submissions/[practice-name].md`
- AuditResult includes: practice profile, detected problems (prioritized by impact), Data4SEO findings, recommended DES services

**Deliverable:** GHL audit form live, `/des-audit` skill working end-to-end, first test AuditResult generated.

---

### Phase 5 — Proposal Engine
**Goal:** Build `/des-proposal` — the core revenue-generating workflow.

**`/des-proposal [contact-id]` skill:**
1. Runs `/des-audit [contact-id]` internally (or loads existing AuditResult if fresh)
2. Loads knowledge base: `company-profile.md`, `services.md`, `market-study.md`, `intelligence/trends.md`
3. Generates personalized HTML proposal:
   - Opens with the practice's specific problems (backed by Data4SEO data)
   - Presents recommended package + à la carte services matched to detected problems
   - Includes ROI math using practice-specific context where possible
   - Competitive comparison section (uses `benchmarking.md`)
   - Implementation roadmap
   - Pricing table
   - CTAs and interactive tabs (matching existing proposal design language)
4. Saves to `proposals/[practice-name]-[date].html`
5. Confirms save location, offers to push to GHL contact record

**Proposal design constraints:**
- Matches the visual quality and structure of `DentalEdgeSolutions_Proposal.html`
- Fully self-contained HTML (no external dependencies)
- Interactive: tabs, CTAs, print/export button
- Mobile-responsive

**Deliverable:** `/des-proposal` skill generating complete, client-ready HTML proposals end-to-end.

---

## Milestone 2: Intelligence & Content Layer

### Phase 6 — Service Portfolio Validation
**Goal:** Update `services.md` based on Market Study findings. Validate that current services map to real problems, identify gaps, flag any pricing adjustments.

**Tasks:**
- Cross-reference Market Study top problems against current service catalog
- Identify problems DES doesn't currently address (opportunities)
- Flag any services that don't map to identified market problems (noise)
- Update service descriptions, target problems, and ideal use cases in `services.md`

**Deliverable:** Validated, updated `services.md`. Any new service ideas documented as proposals, not added to catalog until reviewed.

---

### Phase 7 — Benchmarking
**Goal:** Build a comprehensive DES-vs-competitors analysis.

**Competitors:** Weave, RevenueWell, Adit, MaxAssist, Aron AI, DoctorLogic

**For each competitor:**
- Services offered and gaps
- Pricing model (published vs. opaque)
- SE Florida presence and market share signals
- Technical approach (DIY tool vs. managed service)
- Weaknesses DES can exploit

**Deliverable:** `knowledge/benchmarking.md` with updated competitive table. Proposals auto-pull from this for the competitive comparison section.

---

### Phase 8 — Intelligence Layer (Trends Engine)
**Goal:** Build automated monthly trends monitoring and the `/des-trends` skill.

**Tracked sources (documented in `knowledge/intelligence/sources.md`):**
- Industry publications: ADA News, Dental Economics, Dentistry Today, Dental Products Report
- Competitor sites: blog pages, pricing pages, new service announcements
- Reddit: r/dentistry, r/DentalHygiene, r/smallbusiness
- LinkedIn: public posts from dental industry KOLs and dental practice owners
- YouTube: dental industry channels (technology, practice management)

**`/des-trends [--refresh]` skill:**
- Fetches and synthesizes content from all tracked sources
- Identifies: emerging technologies, regulatory changes, competitive moves, patient behavior shifts, SE Florida-specific signals
- Updates `knowledge/intelligence/trends.md` with dated entries
- Produces 3–5 `Content Briefs` saved to `knowledge/intelligence/briefs/`
- Cadence: monthly automated (scheduled agent) + on-demand manual trigger

**Deliverable:** `/des-trends` skill working, initial `trends.md` populated, first batch of content briefs generated.

---

### Phase 9 — Content Pipeline
**Goal:** Build the `/des-content` skill for trend-triggered and on-demand content generation.

**`/des-content [brief-id | topic]` skill:**
- Accepts a brief ID (from `knowledge/intelligence/briefs/`) or a free-form topic
- Loads relevant knowledge base context (company-profile, services, market-study, trends)
- Generates a complete blog post or service page:
  - Optimized for SE Florida local search
  - Written for practice owners, practice managers, and dentists
  - Includes FAQ schema markup where appropriate (AEO/GEO alignment)
  - Ties content to DES service positioning
- Presents draft for review
- On approval: publishes to GHL blog via GHL MCP

**Deliverable:** `/des-content` skill generating and publishing complete, SEO-ready content to GHL.

---

### Phase 10 — Integration & Polish
**Goal:** Wire all layers together and document the complete platform.

**Tasks:**
- Ensure proposals automatically pull current `trends.md` for trend-relevant insights
- Ensure content briefs reference current `benchmarking.md` for competitive angles
- Add `/des-interview` skill for running structured Company Study updates (re-interview when DES services change)
- Write operator runbook: how to run each skill, expected outputs, troubleshooting
- Write `SKILLS.md`: quick reference for all custom skills and their parameters

**Deliverable:** Fully integrated platform with complete documentation. All five skills working and documented.

---

## Custom Skills — Final Specification

| Skill | Trigger | Inputs | Outputs |
|-------|---------|--------|---------|
| `/des-interview` | Company Study or update | Interactive Q&A | `knowledge/company-profile.md` |
| `/des-audit [contact-id]` | Before proposal generation | GHL contact ID | `audit/submissions/[name].md` |
| `/des-proposal [contact-id]` | New prospect ready | GHL contact ID | `proposals/[name]-[date].html` |
| `/des-trends [--refresh]` | Monthly + on-demand | Tracked sources | Updated `trends.md`, content briefs |
| `/des-content [brief-id\|topic]` | Content needed | Brief or topic | Published GHL blog post |

---

## Data Flows

### Proposal Generation
```
Prospect fills GHL audit form
        ↓
/des-proposal [contact-id]
        ↓
GHL MCP → pulls contact + form data
        ↓
Data4SEO → local rankings + competitor comparison + GBP signals
        ↓
Knowledge Base → company-profile + services + market-study + trends
        ↓
Claude generates HTML proposal
        ↓
Saved to /proposals/[name]-[date].html
        ↓
DES reviews → approves
        ↓
GHL MCP → pushed to contact record
```

### Trends Intelligence
```
Monthly schedule (or /des-trends --refresh)
        ↓
Fetch: publications + competitor sites + Reddit + LinkedIn + YouTube
        ↓
Claude synthesizes → updates trends.md
        ↓
Claude produces 3–5 content briefs → /knowledge/intelligence/briefs/
        ↓
DES reviews briefs → selects topics
        ↓
/des-content [brief-id] → generates post → review → publish to GHL blog
```

---

## Build Sequence & Dependencies

```
Phase 1 (Foundation)
    ├── Phase 2 (Company Study)     ← can start immediately
    └── Phase 3 (Market Study)      ← can start immediately, parallel to Phase 2
            ↓
        Phase 4 (Audit System)      ← requires Phase 3
            ↓
        Phase 5 (Proposal Engine)   ← requires Phases 2, 3, 4
            ↓
        ✅ MILESTONE 1 COMPLETE — /des-proposal working end-to-end
            ↓
        Phase 6 (Portfolio Validation)  ← requires Phases 2, 3
        Phase 7 (Benchmarking)          ← requires Phase 3
        Phase 8 (Trends Engine)         ← requires Phase 1
        Phase 9 (Content Pipeline)      ← requires Phase 8
            ↓
        Phase 10 (Integration & Polish)
            ↓
        ✅ MILESTONE 2 COMPLETE — full platform operational
```

---

## Success Criteria

**Milestone 1 complete when:**
- `/des-proposal [contact-id]` generates a complete, client-ready HTML proposal in under 5 minutes
- Proposal includes practice-specific Data4SEO findings (not generic placeholders)
- Proposal matches the visual quality of `DentalEdgeSolutions_Proposal.html`
- Completed proposal can be pushed to GHL contact record in one command

**Milestone 2 complete when:**
- `/des-trends` refreshes the knowledge base monthly without manual intervention
- `/des-content` generates and publishes SEO-ready content to GHL blog
- Proposals automatically incorporate current trends and updated competitive data
- All five skills are documented and runnable by any DES team member
